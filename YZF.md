# WEB

## model

- 使用框架gorm/jinzhu，viper等

model

```go
package model

import (
	"github.com/jinzhu/gorm"
	"time"
)

/* 用户 table_name = user */
type User struct {
	ID            int           //用户编号
	Name          string        `gorm:"size:32;unique"`  //用户名
	Password_hash string        `gorm:"size:128" `       //用户密码加密的
	Mobile        string        `gorm:"size:11;unique" ` //手机号
	Real_name     string        `gorm:"size:32" `        //真实姓名  实名认证
	Id_card       string        `gorm:"size:20" `        //身份证号  实名认证
	Avatar_url    string        `gorm:"size:256" `       //用户头像路径       通过fastdfs进行图片存储
	Houses        []*House      //用户发布的房屋信息  一个人多套房
	Orders        []*OrderHouse //用户下的订单       一个人多次订单
}

/* 房屋信息 table_name = house */
type House struct {
	gorm.Model                    //房屋编号
	UserId          uint          //房屋主人的用户编号  与用户进行关联
	AreaId          uint          //归属地的区域编号   和地区表进行关联
	Title           string        `gorm:"size:64" `                 //房屋标题
	Address         string        `gorm:"size:512"`                 //地址
	Room_count      int           `gorm:"default:1" `               //房间数目
	Acreage         int           `gorm:"default:0" json:"acreage"` //房屋总面积
	Price           int           `json:"price"`
	Unit            string        `gorm:"size:32;default:''" json:"unit"`               //房屋单元,如 几室几厅
	Capacity        int           `gorm:"default:1" json:"capacity"`                    //房屋容纳的总人数
	Beds            string        `gorm:"size:64;default:''" json:"beds"`               //房屋床铺的配置
	Deposit         int           `gorm:"default:0" json:"deposit"`                     //押金
	Min_days        int           `gorm:"default:1" json:"min_days"`                    //最少入住的天数
	Max_days        int           `gorm:"default:0" json:"max_days"`                    //最多入住的天数 0表示不限制
	Order_count     int           `gorm:"default:0" json:"order_count"`                 //预定完成的该房屋的订单数
	Index_image_url string        `gorm:"size:256;default:''" json:"index_image_url"`   //房屋主图片路径
	Facilities      []*Facility   `gorm:"many2many:house_facilities" json:"facilities"` //房屋设施   与设施表进行关联
	Images          []*HouseImage `json:"img_urls"`                                     //房屋的图片   除主要图片之外的其他图片地址
	Orders          []*OrderHouse `json:"orders"`                                       //房屋的订单    与房屋表进行管理
}

/* 区域信息 table_name = area */ //区域信息是需要我们手动添加到数据库中的
type Area struct {
	Id     int      `json:"aid"`                  //区域编号     1    2
	Name   string   `gorm:"size:32" json:"aname"` //区域名字     昌平 海淀
	Houses []*House `json:"houses"`               //区域所有的房屋   与房屋表进行关联
}

/* 设施信息 table_name = "facility"*/ //设施信息 需要我们提前手动添加的
type Facility struct {
	Id     int      `json:"fid"`     //设施编号
	Name   string   `gorm:"size:32"` //设施名字
	Houses []*House //都有哪些房屋有此设施  与房屋表进行关联的
}

/* 房屋图片 table_name = "house_image"*/
type HouseImage struct {
	Id      int    `json:"house_image_id"`      //图片id
	Url     string `gorm:"size:256" json:"url"` //图片url     存放我们房屋的图片
	HouseId uint   `json:"house_id"`            //图片所属房屋编号
}

/* 订单 table_name = order */
type OrderHouse struct {
	gorm.Model            //订单编号
	UserId      uint      `json:"user_id"`       //下单的用户编号   //与用户表进行关联
	HouseId     uint      `json:"house_id"`      //预定的房间编号   //与房屋信息进行关联
	Begin_date  time.Time `gorm:"type:datetime"` //预定的起始时间
	End_date    time.Time `gorm:"type:datetime"` //预定的结束时间
	Days        int       //预定总天数
	House_price int       //房屋的单价
	Amount      int       //订单总金额
	Status      string    `gorm:"default:'WAIT_ACCEPT'"` //订单状态
	Comment     string    `gorm:"size:512"`              //订单评论
	Credit      bool      //表示个人征信情况 true表示良好
}
```

modelUtils

```go
package model

import (
	"fmt"
	"github.com/gomodule/redigo/redis"
	"github.com/jinzhu/gorm"
	"github.com/spf13/viper"
	"os"
)

//初始化配置, 从vip里取数据
func InitConfig() {
	workDir, err := os.Getwd()
	//workDir := "D:\\My_code\\go\\src\\go_code\\yaozhaofang\\web"
	//workDir, err := filepath.Abs(filepath.Dir(os.Args[0]))
	viper.SetConfigName("application")
	viper.SetConfigType("yml")
	viper.AddConfigPath(workDir + "/conf")
	err = viper.ReadInConfig()
	if err != nil {
		panic(err)
	}
}

//创建数据库连接句柄
var GlobalDB *gorm.DB

//创建redis连接池
var GlobalRedis redis.Pool

func InitDb() error {
	//用viper导入配置文件
	host := viper.GetString("datasource.host")
	port := viper.GetString("datasource.port")
	database := viper.GetString("datasource.database")
	username := viper.GetString("datasource.username")
	password := viper.GetString("datasource.password")
	charset := viper.GetString("datasource.charset")
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=%s&parseTime=true&loc=Local",
		username,
		password,
		host,
		port,
		database,
		charset,
	)

	db, err := gorm.Open("mysql", dsn)
	if err != nil {
		fmt.Println("service user connect mysql err", err)
		return err
	}
	//初始化全局句柄
	//连接池设置
	//设置初始化数据库连接数量
	db.DB().SetMaxIdleConns(50)
	db.DB().SetConnMaxLifetime(100)
	db.DB().SetConnMaxLifetime(60 * 5)

	db.SingularTable(true)

	//默认情况下表名是复数
	GlobalDB = db

	//创建表
	return db.AutoMigrate(new(User), new(House), new(Area), new(Facility), new(HouseImage), new(OrderHouse)).Error

}

//初始化redis链接
func InitRedis() {
	password := viper.GetString("redis.password")
	GlobalRedis = redis.Pool{
		MaxIdle:     20,
		MaxActive:   50,
		IdleTimeout: 60 * 5,
		Dial: func() (redis.Conn, error) {
			return redis.Dial("tcp", "localhost:6379", redis.DialPassword(password))
		},
	}
}
```



## utils

初始化micro封装，错误处理

### utils

```go
//初始化micro
func InitMicro() client.Client{
//	初始化客户端
	consulReg:=consul.NewRegistry()
	microClient := micro.NewService(
		micro.Registry(consulReg),
	)
	return microClient.Client()
}
```

### errmsg

```go
package utils

const (
	RECODE_OK        = "0"    //"成功"
	RECODE_DBERR     = "4001" //"数据库查询错误"
	RECODE_NODATA    = "4002" //"无数据"
	RECODE_DATAEXIST = "4003" //"数据已存在"
	RECODE_DATAERR   = "4004" //"数据错误"

	RECODE_SESSIONERR = "4101" //"用户未登录"
	RECODE_LOGINERR   = "4102" //"用户登录失败"
	RECODE_PARAMERR   = "4103" //"参数错误"
	RECODE_USERONERR  = "4104" //"用户已经注册"
	RECODE_ROLEERR    = "4105" //"用户身份错误"
	RECODE_PWDERR     = "4106" //"密码错误"
	RECODE_USERERR    = "4107" //"用户不存在或未激活"
	RECODE_IMGSERR    = "4108" //"图片验证码错误"
	RECODE_SMSERR     = "4109" //"短信验证码错误"
	RECODE_MOBILEERR  = "4110" //"手机号错误"

	RECODE_REQERR    = "4201" //"非法请求或请求次数受限"
	RECODE_IPERR     = "4202" //"IP受限"
	RECODE_THIRDERR  = "4301" //"第三方系统错误"
	RECODE_IOERR     = "4302" //"文件读写错误"
	RECODE_SERVERERR = "4500" //"内部错误"
	RECODE_UNKNOWERR = "4501" //"未知错误"
)

var recodeText = map[string]string{
	RECODE_OK:         "成功",
	RECODE_DBERR:      "数据库查询错误",
	RECODE_NODATA:     "无数据",
	RECODE_DATAEXIST:  "数据已存在",
	RECODE_DATAERR:    "数据错误",
	RECODE_SESSIONERR: "用户未登录",
	RECODE_LOGINERR:   "用户登录失败",
	RECODE_PARAMERR:   "参数错误",
	RECODE_USERERR:    "用户不存在或未激活",
	RECODE_USERONERR:  "用户已经注册",
	RECODE_ROLEERR:    "用户身份错误",
	RECODE_PWDERR:     "密码错误",
	RECODE_REQERR:     "非法请求或请求次数受限",
	RECODE_IPERR:      "IP受限",
	RECODE_THIRDERR:   "第三方系统错误",
	RECODE_IOERR:      "文件读写错误",
	RECODE_SERVERERR:  "内部错误",
	RECODE_UNKNOWERR:  "未知错误",
	RECODE_IMGSERR:    "图片验证码错误",
	RECODE_SMSERR:     "短信验证码错误",
	RECODE_MOBILEERR:  "手机号错误",
}

//RecodeText 根据errmsg 获取具体信息
func RecodeText(code string) string {
	str, ok := recodeText[code]
	if ok {
		return str
	}
	return recodeText[RECODE_UNKNOWERR]
}

```

## controller

### user

GetImageCd是先调用captcha微服务，获取到验证码内容

```go

// 获取 Session 数据
func GetSession(ctx *gin.Context) {
	resp := make(map[string]interface{})

	s := sessions.Default(ctx) // 初始化 Session 对象
	//从session数据里找userName的数据
	userName := s.Get("userName")

	// 用户没有登录.---没存在 MySQL中, 也没存在 Session 中
	if userName == nil {
		resp["errno"] = utils.RECODE_SESSIONERR
		resp["errmsg"] = utils.RecodeText(utils.RECODE_SESSIONERR)
	} else {
		resp["errno"] = utils.RECODE_OK
		resp["errmsg"] = utils.RecodeText(utils.RECODE_OK)

		var nameData struct {
			Name string `json:"name"`
		}
		nameData.Name = userName.(string) // 类型断言
		fmt.Println(nameData)
		resp["data"] = nameData
	}

	ctx.JSON(http.StatusOK, resp)
}

//获取验证码图片信息，从consul里调用微服务
func GetImageCd(ctx *gin.Context) {
	uuid := ctx.Param("uuid")

	consulService := utils.InitMicro()
	//调用对应service的Call函数
	//我们这边看到的是NewGetCaptchaService(服务名，服务发现)
	microClient := getCaptcha.NewGetCaptchaService("go.micro.srv.getCaptcha", consulService)

	//客户端初始化了，可以调用getCaptcha函数的方法
	resp, err := microClient.MicroGetCaptcha(context.TODO(), &getCaptcha.Request{Uuid: uuid})
	if err != nil {
		fmt.Println("microClient.Cal err:", err)
	}

	//用captcha模块的图片
	var img captcha.Image
	//拿到字节流，反序列化成图片
	json.Unmarshal(resp.Img, &img)

	//将图片解码到浏览器
	png.Encode(ctx.Writer, img)
}

func GetSmsCd(ctx *gin.Context) {
	//获取手机号
	mobile := ctx.Param("mobile")
	//拆分GET请求中的URL格式==格式：资源路径?k=v&k=v&k=v
	text := ctx.Query("text")
	uuid := ctx.Query("id")

	//校验手机号格式
	reg, _ := regexp.Compile(`^1[3,4,5,7,8]\d{9}$`)
	isRightMobile := reg.MatchString(mobile)
	if !isRightMobile {
		log.Fatal("手机号格式错误")
		return
	}

	if mobile == "" || text == "" || uuid == "" {
		log.Fatal("GetSmsCd传入数据不完整")
		return
	}

	consulService := utils.InitMicro()

	//初始化客户端
	microClient := register.NewRegisterService("go.micro.srv.register", consulService)

	//调用远程函数
	resp, err := microClient.SmsCode(context.TODO(), &register.Request{
		Uuid:   uuid,
		Text:   text,
		Mobile: mobile,
	})
	if err != nil {
		fmt.Println("microClient.SendSms err:", err)
		return
	}

	//写入校验结果在浏览器
	ctx.JSON(http.StatusOK, resp)
}

//发送注册信息
func PostRet(ctx *gin.Context) {
	type RegisterUser struct {
		Mobile   string `json:"mobile"`
		Password string `json:"password"`
		SmsCode  string `json:"sms_code"`
	}
	//确定容器
	resp := make(map[string]interface{})
	//绑定数据
	var regUser RegisterUser
	err := ctx.Bind(&regUser)
	if err != nil {
		resp["errno"] = utils.RECODE_DATAERR
		resp["errmsg"] = utils.RecodeText(utils.RECODE_DATAERR)
		ctx.JSON(200, resp)
		return
	}
	//校验数据
	if regUser.Mobile == "" || regUser.Password == "" || regUser.SmsCode == "" {
		resp["errno"] = utils.RECODE_DATAERR
		resp["errmsg"] = utils.RecodeText(utils.RECODE_DATAERR)
		ctx.JSON(200, resp)
		return
	}
	//正则校验手机号
	reg, _ := regexp.Compile(`^1[3,4,5,7,8]\d{9}$`)
	if !reg.MatchString(regUser.Mobile) {
		resp["errno"] = utils.RECODE_DATAERR
		resp["errmsg"] = utils.RecodeText(utils.RECODE_DATAERR)
		ctx.JSON(200, resp)
		return
	}
	//远程调用
	//初始化客户端
	consulService := utils.InitMicro()

	//初始化客户端
	microClient := register.NewRegisterService("go.micro.srv.register", consulService)

	//调用远程函数
	response, err := microClient.Register(context.TODO(), &register.RegRequest{
		Mobile:   regUser.Mobile,
		SmsCode:  regUser.SmsCode,
		Password: regUser.Password,
	})
	if err != nil {
		resp["errno"] = utils.RECODE_DATAERR
		resp["errmsg"] = utils.RecodeText(utils.RECODE_DATAERR)
		ctx.JSON(200, resp)
		return
	}

	if response.Errno == utils.RECODE_OK {
		s := sessions.Default(ctx)
		s.Set("userName", regUser.Mobile)
		s.Save()
	}
}

// 测试实现：
// 获取地域信息的微服务
func GetArea(ctx *gin.Context) {

	//调用远程逻辑获取地域信息
	microClient := getArea.NewGetAreaService("go.micro.srv.getArea", utils.InitMicro())

	//连接传参
	resp, err := microClient.MicroGetArea(context.TODO(), &getArea.Request{})
	if err != nil {
		fmt.Println(err)
	}

	ctx.JSON(http.StatusOK, resp)

}

//登录逻辑调用微服务
func PostLogin(ctx *gin.Context) {
	//	获取前端数据
	var loginData struct {
		Mobile   string `json:"mobile"`
		PassWord string `json:"password"`
	}
	err := ctx.Bind(&loginData)
	if err != nil {
		fmt.Println("PostLogin获取数据失败")
		return
	}

	//初始化客户端
	//把登录放在微服务了
	microClient := register.NewRegisterService("go.micro.srv.register", utils.InitMicro())

	//调用远程服务
	resp, err := microClient.Login(context.TODO(), &register.RegRequest{
		Mobile:   loginData.Mobile,
		Password: loginData.PassWord,
	})
	defer ctx.JSON(http.StatusOK, resp)
	if err != nil {
		fmt.Println("调用login服务错误", err)
		return
	}
	//返回数据  存储session  并返回数据给web端
	session := sessions.Default(ctx)
	session.Set("userName", resp.Name)
	session.Save()

}

// 退出登录不用微服务,正常逻辑删除session即可
func DeleteSession(ctx *gin.Context) {
	resp := make(map[string]interface{})

	// 初始化 Session 对象
	s := sessions.Default(ctx)
	// 删除 Session 数据
	s.Delete("userName") // 没有返回值
	// 必须使用 Save 保存
	err := s.Save() // 有返回值

	if err != nil {
		resp["errno"] = utils.RECODE_IOERR // 没有合适错误,使用 IO 错误!
		resp["errmsg"] = utils.RecodeText(utils.RECODE_IOERR)

	} else {
		resp["errno"] = utils.RECODE_OK
		resp["errmsg"] = utils.RecodeText(utils.RECODE_OK)
	}
	ctx.JSON(http.StatusOK, resp)
}

// 获取用户基本信息,调用user/MicroGetUser微服务实现
func GetUserInfo(ctx *gin.Context) {
	s := sessions.Default(ctx)    // Session 初始化
	userName := s.Get("userName") // 根据key 获取Session

	//调用user/MicroGetUser微服务
	microClient := user.NewUserService("go.micro.srv.user", utils.InitMicro())

	resp, err := microClient.MicroGetUser(context.TODO(), &user.Request{
		Name: userName.(string),
	})
	if err != nil {
		fmt.Println("web/controller/user GetUserInfo调用微服务出错", err)
		resp.Errno = utils.RECODE_DATAERR
		resp.Errmsg = utils.RecodeText(utils.RECODE_DATAERR)
	}

	ctx.JSON(http.StatusOK, resp)

}

//修改用户名
func PutUserInfo(ctx *gin.Context) {
	//	获取当前的用户名
	s := sessions.Default(ctx) //初始化session对象
	userName := s.Get("userName")

	//	获取新的用户名
	var nameData struct {
		Name string `json:"name"`
	}
	err := ctx.Bind(&nameData)
	if err != nil {
		fmt.Println("web/controller/user PutUserInfo 获取数据 err", err)
	}

	//调用微服务UpdateUserName
	microClient := user.NewUserService("go.micro.srv.user", utils.InitMicro())

	//调用远程服务
	resp, _ := microClient.UpdateUserName(context.TODO(), &user.UpdateReq{
		NewName: nameData.Name,
		OldName: userName.(string),
	})

	//更新session数据
	if resp.Errno == utils.RECODE_OK {
		s.Set("userName", nameData.Name)
		s.Save()
	}
	ctx.JSON(http.StatusOK, resp)
}

//校验用户真实姓名
func PutUserAuth(ctx *gin.Context) {
	//	获取用户数据
	type AuthStu struct {
		IdCard   string `json:"id_card"`
		RealName string `json:"real_name"`
	}
	var auth AuthStu
	err := ctx.Bind(&auth)
	//	校验数据
	if err != nil {
		fmt.Println("获取数据错误", err)
		return
	}

	//取出session数据
	session := sessions.Default(ctx)

	userName := session.Get("userName")

	//	处理数据的微服务
	microClient := user.NewUserService("go.micro.srv.user", utils.InitMicro())
	//调用远程服务
	resp, _ := microClient.AuthUpdate(context.TODO(), &user.AuthReq{
		UserName: userName.(string),
		RealName: auth.RealName,
		IdCard:   auth.IdCard,
	})
	ctx.JSON(http.StatusOK, resp)
}
```



# service

## getArea

proto

```protobuf
syntax = "proto3";

package go.micro.srv.getArea;
option go_package="./;getArea";

service GetArea {
	rpc MicroGetArea(Request) returns (Response) {}
}

message Request {
}

message Response {
	string errno = 1;
	string errmsg = 2;
	repeated AreaInfo data = 3;
}

message AreaInfo{
	int32 aid = 1;
	string aname = 2;
}
```

main

```go
package main

import (
	"github.com/micro/go-micro"
	"github.com/micro/go-micro/registry"
	"github.com/micro/go-plugins/registry/consul"
	"go_code/yaozhaofang/service/getArea/handler"
	"go_code/yaozhaofang/service/getArea/model"
	"log"

	getArea "go_code/yaozhaofang/service/getArea/proto/getArea"
)

func main() {
	err := model.InitDb()
	if err != nil {
		log.Fatalf("model.InitDb() err = %v", err)
	}
	model.InitRedis()
	//配置注册consul的地址
	regOpt := registry.Option(func(options *registry.Options){
			options.Addrs= []string{"172.17.0.1"}
	})
	consulRegistry := consul.NewRegistry(regOpt)


	// New Service 监听的地址
	service := micro.NewService(
		micro.Address("localhost:52668"),
		micro.Name("go.micro.srv.getArea"),
		micro.Registry(consulRegistry),
		micro.Version("latest"),
	)

	// Initialise service
	service.Init()

	// Register Handler
	getArea.RegisterGetAreaHandler(service.Server(), new(handler.GetArea))

	// Run service
	if err := service.Run(); err != nil {
		log.Fatal(err)
	}
}
```



## getCaptcha

proto

```protobuf
syntax = "proto3";

package go.micro.srv.getCaptcha;
option go_package="./;getCaptcha";
service GetCaptcha {
	rpc MicroGetCaptcha(Request) returns (Response) {}
}

message Request {
	string uuid = 1;
}

message Response {
	string errno = 1;
	string errmsg = 2;
//	使用切片存储图片信息，用json序列化
	bytes img=3;
}
```

handler

- 生成图验的框架，需要字体的支持

```
"github.com/afocus/captcha"
```

- 设置好图片 格式，将图片验证码的内容存入redis（setex 5分钟 uuid : 内容）， 图片传输到前端
- uuid由前端的代码生成

![image-20220307003723891](http://myimg.go2flare.xyz/img/image-20220307003723891.png)

```go

type GetCaptcha struct{}

func (e *GetCaptcha) MicroGetCaptcha(ctx context.Context, req *getCaptcha.Request, rsp *getCaptcha.Response) error {
	log.Log("Received GetCaptcha.Call request")
	//	初始化对象
	cap := captcha.New()
	//	设置字体
	cap.SetFont("./conf/comic.ttf")

	//	设置验证码大小
	cap.SetSize(128, 64)
	//	设置干扰强度
	cap.SetDisturbance(captcha.MEDIUM)
	//	设置前景色
	cap.SetFrontColor(color.RGBA{255, 245, 247, 255})
	//	设置背景色
	cap.SetBkgColor(color.RGBA{101, 147, 74, 128}, color.RGBA{69, 137, 148, 255},
		color.RGBA{255, 150, 125, 255})
	//	生成字体 ,将验证码展示到网页
	//http.HandleFunc("/r", func(w http.ResponseWriter, r *http.Request) {
	img, str := cap.Create(4, captcha.NUM)
	//存储图片验证码str到redis
	err := model.SaveImgCode(str, req.Uuid)
	if err != nil{
		rsp.Errno = utils.RECODE_DBERR
		rsp.Errmsg = utils.RecodeText(utils.RECODE_DBERR)
		return err
	}

	rsp.Errno = utils.RECODE_OK
	rsp.Errmsg = utils.RecodeText(utils.RECODE_OK)
	//生成的图片序列化
	imgBuf, err := json.Marshal(img)
	if err != nil {
		fmt.Println("json.Marshal err", err)
		return err
	}
	//imgBuf用rsp通过消息体定义的img传出
	rsp.Img = imgBuf

	return nil
}
```



这里可能需要将短信验证码和图片验证码合并到一起

```

```

main

```go
package main

import (
	"github.com/micro/go-micro"
	"github.com/micro/go-micro/registry"
	"github.com/micro/go-micro/util/log"
	"github.com/micro/go-plugins/registry/consul"
	//"github.com/micro/protoc-gen-micro/plugin/micro"
	"go_code/yaozhaofang/service/getCaptcha/handler"
	"go_code/yaozhaofang/service/getCaptcha/model"
	getCaptcha "go_code/yaozhaofang/service/getCaptcha/proto/getCaptcha"
)

func main() {
	//初始化consul注册服务
	regOpt := registry.Option(func(options *registry.Options){
		options.Addrs= []string{"172.17.0.1"}
	})
	consulRegistry := consul.NewRegistry(regOpt)

	//初始化reds
	model.InitRedis()

	// New Service
	service := micro.NewService(
		//指定微服务连接服务发现的端口
		micro.Address("localhost:52666"), //防止随机生成port
		micro.Name("go.micro.srv.getCaptcha"),
		micro.Registry(consulRegistry),
		micro.Version("latest"),
	)

	// Initialise service
	service.Init()

	// Register Handler
	getCaptcha.RegisterGetCaptchaHandler(service.Server(), new(handler.GetCaptcha))

	// Run service
	if err := service.Run(); err != nil {
		log.Fatal(err)
	}
}
```

## house

proto

```protobuf
syntax = "proto3";

package go.micro.srv.house;
option go_package="./;house";

service House {
	rpc PubHouse(Request) returns (Response) {};
	rpc UploadHouseImg(ImgReq)returns(ImgResp){};
	rpc GetHouseInfo(GetReq)returns(GetResp){};
	rpc GetHouseDetail(DetailReq)returns(DetailResp){};
	rpc GetIndexHouse(IndexReq)returns(GetResp){};
	rpc SearchHouse(SearchReq)returns(GetResp){};
}

message SearchReq{
	string aid = 1;
	string sd = 2;
	string ed = 3;
	string sk = 4;
}

message IndexReq{
}


message DetailReq{
	string houseId = 1;
	string userName = 2;
}

message DetailResp{
	string errno = 1;
	string errmsg = 2;
	DetailData data = 3;
}

message DetailData{
	HouseDetail house = 1;
	int32 user_id = 2;
}

message HouseDetail{
	int32 acreage = 1;
	string address = 2;
	string beds = 3;
	int32 capacity = 4;
	//comment
	repeated CommentData comments = 5;
	int32 deposit=6;
	//展示所有的图片 主图片和副图片
	repeated int32 facilities = 7; //家具id切片
	int32 hid = 8;
	repeated string img_urls = 9;
	int32 max_days = 10;
	int32 min_days = 11;
	int32 price = 12;
	int32 room_count = 13;
	string title = 14;
	string unit = 15;
	string user_avatar = 16;
	int32 user_id = 17;
	string user_name = 18;
}

message CommentData{
	string comment = 1;
	string ctime = 2;
	string user_name = 3;
}


message GetReq{
	string userName = 1;
}

message GetResp{
	string errno = 1;
	string errmsg = 2;
	GetData data = 3;
}

message GetData{
	repeated Houses houses = 1;
}

message Houses {
	string address = 1;
	string area_name = 2;
	string ctime = 3;
	int32 house_id = 4;
	string img_url = 5;
	int32 order_count = 6;
	int32 price = 7;
	int32 room_count = 8;
	string title = 9;
	string user_avatar = 10;
}

message ImgReq{
	string houseId = 1;
	bytes imgData = 2;
	string fileExt = 3;
}

message ImgResp{
	string errno = 1;
	string errmsg = 2;
	ImgData data = 3;
}

message ImgData{
	string url = 1;
}

message Request {
	string acreage = 1;
	string address = 2;
	string area_id = 3;
	string beds = 4;
	string capacity = 5;
	string deposit = 6;
	repeated string facility = 7;
	string max_days = 8;
	string min_days = 9;
	string price = 10;
	string room_count = 11;
	string title = 12;
	string unit = 13;
	string userName = 14;
}

message Response {
	string errno = 1;
	string errmsg = 2;
	HouseData data = 3;
}

message HouseData{
	string house_id = 1;
}
```

main

```go
package main

import (
	"github.com/micro/go-micro"
	"github.com/micro/go-micro/registry"
	"github.com/micro/go-plugins/registry/consul"
	"go_code/yaozhaofang/service/house/handler"
	"go_code/yaozhaofang/service/house/model"
	"log"

	house "go_code/yaozhaofang/service/house/proto/house"
)

func main() {
	model.InitDb()

	regOpt := registry.Option(func(options *registry.Options){
		options.Addrs= []string{"172.17.0.1"}
	})
	consulRegistry := consul.NewRegistry(regOpt)

	// New Service
	service := micro.NewService(
		micro.Address("localhost:52671"),
		micro.Name("go.micro.srv.house"),
		micro.Registry(consulRegistry),
		micro.Version("latest"),
	)

	// Initialise service
	service.Init()

	// Register Handler
	house.RegisterHouseHandler(service.Server(), new(handler.House))

	// Run service
	if err := service.Run(); err != nil {
		log.Fatal(err)
	}
}
```



## register

```protobuf
syntax = "proto3";

package go.micro.srv.register;
option go_package="./;register";

service Register {
	rpc SmsCode(Request) returns (Response) {}
	rpc Register(RegRequest) returns (RegResponse) {}
	rpc Login(RegRequest) returns (RegResponse) {};
}

message RegRequest{
	string mobile = 1;
	string password = 2;
	string sms_code = 3;
}

//添加session,注册之后直接是登录状态
message RegResponse{
	string errno = 1;
	string errmsg = 2;
	string name = 3;
}


message Request {
	string mobile = 1;
	string text = 2;
	string uuid = 3;
}

message Response {
	string errno = 1;
	string errmsg = 2;
}

```

### 基本逻辑

- 获取到图片验证码的uuid，从redis取出，和前端输入的图验内容比对
- 调用阿里云API发送短信验证码，将短信验证码存入redis（setex, 手机号，5分钟，短信内容），与前端输入的短验内容比对
- 将密码使用sha256 hash加密，存储string转16进制

main

```go
package main

import (
	"github.com/micro/go-micro"
	"github.com/micro/go-micro/registry"
	"github.com/micro/go-micro/util/log"
	"github.com/micro/go-plugins/registry/consul"
	//"github.com/micro/protoc-gen-micro/plugin/micro"
	"go_code/yaozhaofang/service/register/handler"
	"go_code/yaozhaofang/service/register/model"
	register "go_code/yaozhaofang/service/register/proto/register"
)

func main() {
	//初始化数据库配置
	model.InitConfig()
	//初始化mysql,redis
	model.InitRedis()
	model.InitDb()

	//服务发现consul
	regOpt := registry.Option(func(options *registry.Options){
		options.Addrs= []string{"172.17.0.1"}
	})
	consulRegistry := consul.NewRegistry(regOpt)

	// New Service
	service := micro.NewService(
		micro.Address("localhost:52669"),
		micro.Name("go.micro.srv.register"),
		micro.Version("lastest"),
		micro.Registry(consulRegistry),
	)

	// Initialise service
	service.Init()

	// Register Handler
	register.RegisterRegisterHandler(service.Server(), new(handler.Register))

	if err := service.Run(); err != nil {
		log.Fatal(err)
	}
}

```



## user

proto

```protobuf
syntax = "proto3";

package go.micro.srv.user;
option go_package="./;user";
service User {
	rpc MicroGetUser(Request) returns (Response) {};
	rpc UpdateUserName(UpdateReq) returns (UpdateResp){};
	rpc UploadAvatar(UploadReq) returns (UploadResp){};
	rpc AuthUpdate(AuthReq) returns (AuthResp){};
}


message AuthReq{
	string id_card = 1;
	string real_name = 2;
	string userName = 3;
}

message AuthResp{
	string errno = 1;
	string errmsg = 2;
}


message UploadData{
	string avatar_url = 1;
}

message UploadResp{
	string errno = 1;
	string errmsg = 2;
	UploadData data = 3;
}

message UploadReq{
	bytes avatar = 1;
	string userName = 2;
	string fileExt = 3;
}


message UpdateReq{
	string newName = 1;
	string oldName = 2;
}

message UpdateResp{
	string errno = 1;
	string errmsg = 2;
	NameData data = 3;
}

message NameData{
	string name = 1;
}

message Request {
	string name = 1;
}

message Response {
	string errno = 1;
	string errmsg = 2;
	UserInfo data = 3;
}

message UserInfo{
	int32 user_id = 1;
	string name = 2;
	string mobile = 3;
	string real_name = 4;
	string id_card = 5;
	string avatar_url = 6;
}
/*
service User {
	rpc SendSms(Request) returns (Response) {};
	rpc Register(RegReq) returns (RegResp) {};
}


message RegReq{
	string mobile = 1;
	string password = 2;
	string sms_code = 3;
}

message RegResp{
	string errno = 1;
	string errmsg = 2;
}

message Request {
	string phone = 1;
	string imgCode = 2;
	string uuid = 3;
}

message Response {
	string errno = 1;
	string errmsg = 2;
}
*/
```

main

```go
package main

import (
	"github.com/micro/go-micro"
	"github.com/micro/go-micro/registry"
	"github.com/micro/go-micro/util/log"
	"github.com/micro/go-plugins/registry/consul"
	"go_code/yaozhaofang/service/user/handler"
	"go_code/yaozhaofang/service/user/model"
	user "go_code/yaozhaofang/service/user/proto/user"
)

func main() {
	//初始化配置
	model.InitConfig()
	//初始化redis连接池
	model.InitRedis()

	//初始化服务发现
	regOpt := registry.Option(func(options *registry.Options){
		options.Addrs= []string{"172.17.0.1"}
	})
	consulRegistry := consul.NewRegistry(regOpt)

	// New Service
	service := micro.NewService(
		micro.Address("localhost:52667"),  //固定端口
		micro.Name("go.micro.srv.user"),
		micro.Registry(consulRegistry),   //注册服务
		micro.Version("latest"),
	)
	// Initialise service
	service.Init()
	//在这里用配置文件初始化
	model.InitDb()

	// Register Handler
	user.RegisterUserHandler(service.Server(), new(handler.User))

	// Run service
	if err := service.Run(); err != nil {
		log.Fatal(err)
	}
	//model.GlobalDB.Close()
}
```



## userOrder

```protobuf
syntax = "proto3";

package go.micro.srv.useOrder;
option go_package="./;userOrder";
service UserOrder {
	rpc CreateOrder(Request) returns (Response) {};
	rpc GetOrderInfo(GetReq)returns(GetResp){};
	rpc UpdateStatus(UpdateReq)returns(UpdateResp){};
}

message UpdateReq{
	string action = 1;
	string reason = 2;
	string id = 3;
}

message UpdateResp{
	string errno = 1;
	string errmsg = 2;
}


message GetReq{
	string role = 1;
	string userName = 2;
}

message GetResp{
	string errno = 1;
	string errmsg = 2;
	GetData data = 3;
}


message GetData{
	repeated OrdersData orders = 1;
}

message OrdersData{
	int32 amount = 1;
	string comment = 2;
	string ctime = 3;
	int32 days = 4;
	string end_date = 5;
	string img_url = 6;
	int32 order_id = 7;
	string start_date = 8;
	string status = 9;
	string title = 10;
}

message Request {
	string house_id = 1;
	string start_date = 2;
	string end_date = 3;
	string userName = 4;
}

message Response {
	string errno = 1;
	string errmsg = 2;
	OrderData data = 3;
}

message OrderData{
	string order_id = 1;
}
```

main

```go
package main

import (
	"github.com/micro/go-micro"
	"github.com/micro/go-micro/registry"
	"github.com/micro/go-micro/util/log"
	"github.com/micro/go-plugins/registry/consul"
	//"github.com/micro/protoc-gen-micro/plugin/micro"
	"go_code/yaozhaofang/service/userOrder/handler"
	"go_code/yaozhaofang/service/userOrder/model"
	userOrder "go_code/yaozhaofang/service/userOrder/proto/userOrder"
)

func main() {
	model.InitConfig()
	model.InitDb()

	regOpt := registry.Option(func(options *registry.Options){
		options.Addrs= []string{"172.17.0.1"}
	})
	consulRegistry := consul.NewRegistry(regOpt)

	// New Service
	service := micro.NewService(
		micro.Address("localhost:52670"),
		micro.Name("go.micro.srv.userOrder"),
		micro.Registry(consulRegistry),
		micro.Version("latest"),
	)

	// Initialise service
	service.Init()

	// Register Handler
	userOrder.RegisterUserOrderHandler(service.Server(), new(handler.UserOrder))
    
	// Run service
	if err := service.Run(); err != nil {
		log.Fatal(err)
	}
}
```



# 遇到的问题

## win开启微服务报错

```
“尝试以访问权限禁止的方式访问套接字”
2022-03-06 15:12:06.580342 I | listen tcp 127.0.0.1:52668: bind: An attempt was made to access a socket in a way forbidden by its access permission
s.
exit status 1

```

ref:

https://appuals.com/fix-an-attempt-was-made-to-access-a-socket-in-a-way-forbidden-by-its-access-permissions/#:~:text=Some%20affected%20users%20have%20discovered,involved%20localhost%20connections%20to%20fail.

这篇文章有详细解决方案

我应该是重启了IIS服务就可以运行了

1. 按

   Windows 键 + R

   打开运行对话框。然后，键入“ 

   cmd

    ”并按

   Ctrl + Shift + Enter

   打开提升的命令提示符窗口。

   ![运行对话框： cmd ，然后按 Ctrl + Shift + Enter](http://myimg.go2flare.xyz/img/cmd.jpg.webp)

   运行对话框： cmd ，然后按 Ctrl + Shift + Enter

2. 在提升的命令提示符内，键入以下命令并按 Enter 重新启动 Internet 信息服务：

   ```
   iis重置
   ```

3. 等到 Internet 服务成功停止并重新启动，然后重复触发错误的相同过程以查看问题是否已解决。

   ![重新启动 Internet 信息服务](http://myimg.go2flare.xyz/img/iireset.jpg.webp)

   重新启动 Internet 信息服务

## redis使用密码连接

在连接redis的时候，使用以下依赖包连接redis

> ```
> "github.com/gomodule/redigo/redis"
> ```

需要传密码的参数，而使用viper的时候，必须要初始化viper的配置，否则会报以下错误

```
2022/03/06 17:50:06 err : NOAUTH Authentication required.
```

```go
func InitConfig() {
	workDir, err := os.Getwd()
	//workDir := "D:\\My_code\\go\\src\\go_code\\yaozhaofang\\web"
	//workDir, err := filepath.Abs(filepath.Dir(os.Args[0]))
	viper.SetConfigName("application")
	viper.SetConfigType("yml")
	viper.AddConfigPath(workDir+"/conf")
	err = viper.ReadInConfig()
	if err != nil {
		panic(err)
	}
}
//初始化redis链接
func InitRedis() {
	GlobalRedis = redis.Pool{
		MaxIdle:     20,
		MaxActive:   50,
		IdleTimeout: 60 * 5,
		Dial: func() (redis.Conn, error) {
			return redis.Dial("tcp", "localhost:6379", redis.DialPassword(viper.GetString("redis.password")))
		},
	}
	fmt.Println(viper.GetString("redis.password"))
}
func TestRedis(t *testing.T){
	InitConfig()//必须要先初始化viper的配置
	InitRedis()
	conn := GlobalRedis.Get()
	byteLen, err := redis.Bytes(conn.Do("SET", "yoyo", "HAHAHa"))
	fmt.Println("存储长度 ：", byteLen)
	if err != nil{
		log.Fatalf("err : %v", err)
	}
}
```

第二种使用gin中的插件

> ```
> "github.com/gin-contrib/sessions/redis"
> ```

错误，参数中使用空指针的错误

```
runtime error: invalid memory address or nil pointer dereference
```

也是因为输错密码，无法操作redis导致

## 实例

### redis操作

utils

```go
//初始化配置, 只要适用到viper都要先初始化
func InitConfig() {
	workDir, err := os.Getwd()
	//workDir := "D:\\My_code\\go\\src\\go_code\\yaozhaofang\\web"
	//workDir, err := filepath.Abs(filepath.Dir(os.Args[0]))
	viper.SetConfigName("application")
	viper.SetConfigType("yml")
	viper.AddConfigPath(workDir)
	viper.AddConfigPath(workDir+"/conf")
	err = viper.ReadInConfig()
	if err != nil {
		panic(err)
	}
}

var (
	//创建数据库连接句柄
	GlobalDB *gorm.DB
	//创建redis连接池
	GlobalRedis redis.Pool
)
//初始化redis链接
func InitRedis() {
	InitConfig()
	GlobalRedis = redis.Pool{
		MaxIdle:     20,
		MaxActive:   50,
		IdleTimeout: 60 * 5,
		Dial: func() (redis.Conn, error) {
			return redis.Dial("tcp", "localhost:6379", redis.DialPassword(viper.GetString("redis.password")))
		},
	}
}
```

test

```
func TestRedis(t *testing.T){
	InitRedis()
	//初始化一条连接
	conn := GlobalRedis.Get()
	bytes, err := redis.Bytes(conn.Do("SET", "yoyo", "shit"))
	fmt.Println("存储得到的redis响应 ：", string(bytes))
	bytes, err = redis.Bytes(conn.Do("GET", "yoyo"))
	fmt.Println("get的数据为 ：", string(bytes))
	if err != nil{
		log.Fatalf("err : %v", err)
	}
	store, _ := ginRedis.NewStore(10, "tcp", "localhost:6379", "", []byte("itcast"))

	//	使用容器
	sessions.Sessions("mysession", store)
}
```

redis使用session实例

```go
func main() {
	router := gin.Default()

//	sessions中间件的redis插件，连接后可以将session对象存储在redis中， key:sessionID, value:session对象
	store, err := redis.NewStore(10, "tcp", "localhost:6379", "4.234.23123", []byte("abcdefg"))
	if err != nil {
		log.Fatalf("redis.NewStore err := %v\n",err)
	}

	//使用session，调用session中间件（session命名，redis的连接句柄）
	router.Use(sessions.Sessions("mysession", store))

	router.GET("/test", func(context *gin.Context){
	//	调用session, 设置session数据
		s:=sessions.Default(context)
		//第一次获取，设置session
		//s.Set(11111, 22222)
		//s.Save()
		//--------
		//第二次获取，直接get session
		v:= s.Get(11111)
		fmt.Println("获取的Session",v.(int))
		context.Writer.WriteString("测试Session....")
	})

	router.Run(":5266")
}
```

# CICD流程

## 一次失败的cicd

### 使用多个dockerfile意味着需要有多个gitlabci job拉取代码

- 我在每个微服务中都编写了dockerfile
- 然后gitlabci，有很多个job，都要拉取所有的代码，不清楚是要一次性拉很多个仓库才能ci吗，所以不能并行执行pineline

## 首先web打包docker镜像

## 微服务打包镜像

## 上传gitlab

## 编写CI/CD yaml文件

### 上传镜像到仓库

### 在使用docker的服务器拉取镜像

### 删除原先的镜像

### 运行新的镜像

### 尝试使用编排工具优化



## 容器无法连接宿主机mysql，redis

```
service user connect mysql err Error 1130: Host '172.17.0.4' is not allowed
```

```
server:
  ip: localhost
  port: 9000

datasource:
  driverName: mysql
  host: 172.17.0.1
  port: 3306
  database: search_house
  username: root
  password: 4.234.23123aa
  charset: utf8

redis:
  network: tcp
  address: 172.17.0.1:6379
  address: 47.106.87.191:6379可用
  password: 4.234.23123
```

### mysql配置

```
配置root用户可被所有用户连接
```

宿主机的ip如何通过容器获取

mysql和redis中配置有连接限制

ifconfig查看网卡

```
br-6d468c5c7427
172.18.0.1 
2022/03/12 07:03:49 mysql配置连接失败，username=root, password=4.234.23123aa, host=172.18.0.1, port=3306, database=test, charset=utf8
2022/03/12 07:03:49 gorm.Open : Error 1130: Host '172.17.0.5' is not allowed to connect to this MySQL server

docker0
172.17.0.1 
2022/03/12 07:09:01 mysql配置连接失败，username=root, password=4.234.23123, host=172.17.0.1, port=3306, database=test, charset=utf8
2022/03/12 07:09:01 gorm.Open : Error 1130: Host '172.17.0.5' is not allowed to connect to this MySQL server

eth0
172.29.113.151
2022/03/12 07:09:52 mysql配置连接失败，username=root, password=4.234.23123aa, host=172.29.113.151, port=3306, database=test, charset=utf8
2022/03/12 07:09:52 gorm.Open : Error 1130: Host '172.17.0.5' is not allowed to connect to this MySQL server
```

我们需要修改mysql 数据库中，user表的root用户的可连接主机，改成所有即可



经过探测连接的程序

```
2022/03/12 09:21:13 mysql配置连接成功，username=root, password=4.234.23123aa, host=172.17.0.1, port=3306, database=test, charset=utf8
```



## 无法连接宿主机consul

```shell
microClient.Cal err: {"id":"go.micro.client","code":500,"detail":"error selecting go.micro.srv.getCaptcha node: Get \"http://127.0.0.1:8500/v1/health/service/go.micro.srv.getCaptcha?stale=\": dial tcp 127.0.0.1:8500: connect: connection refused","status":"Internal Server Error"}
```

```go
	//配置注册consul的地址
	regOpt := registry.Option(func(options *registry.Options){
			options.Addrs= []string{"172.17.0.1"}
	})
	consulRegistry := consul.NewRegistry(regOpt)
```

 

