# 基于k8s项目的gorm

[文档](https://v1.gorm.io/docs/)

版本github.com/jinzhu/gorm v1.9.16

# 数据库连接

## 连接配置逻辑

> database.go

初始化配置

```go
//创建数据库连接句柄
var (
	GlobalDB *gorm.DB
	GlobalConfig *DBConfig
	log = logi.Log.Sugar()
)

//数据库配置项
type DBConfig struct {
	DBType string // 数据库类型，默认mysql
	DSN    string // data source name

	MaxIdleConns int
	MaxOpenConns int
	AutoMigrate  bool // 自动建表，补全缺失字段，初始化数据
	Debug        bool
	CacheFlag    bool

	CacheExpiration      time.Duration
	CacheCleanupInterval time.Duration

	// Deprecated: use DSN instead. TODO 向后兼容，若干个版本后删掉
	DBUser, DBPassword, DBAddr, DBName string
}

//默认配置
func defaultDbConfig(cfg *DBConfig) *DBConfig {
	newCfg := *cfg
	if newCfg.DBType == ""{
		newCfg.DBType = "mysql"
	}

	if newCfg.MaxIdleConns == 0{
		newCfg.MaxIdleConns = 10
	}

	if newCfg.MaxOpenConns == 0{
		newCfg.MaxOpenConns = 20
	}

	if newCfg.MaxOpenConns == 0{
		newCfg.MaxOpenConns = 20
	}

	if newCfg.CacheExpiration == 0 {
		newCfg.CacheExpiration = 5 * time.Minute
	}

	if newCfg.CacheCleanupInterval == 0 {
		newCfg.CacheCleanupInterval = 10 * time.Minute
	}
	return &newCfg
}
//获取初始化数据库配置对象
func InitDataBase(cfg *DBConfig) {
	//最后一个子串的下标截取登录的账号信息
	slashIndex := strings.LastIndex(cfg.DSN, "/")
	dsn := cfg.DSN[:slashIndex+1]
	dbName := cfg.DSN[slashIndex+1:]
	dsn = fmt.Sprintf("%s?charset=utf8mb4&parseTime=True&loc=Local",dsn)
	db, err := sql.Open("mysql", dsn)
	if err != nil{
		panic(err)
	}
	defer db.Close()

	createSQL := fmt.Sprintf(
		"CREATE DATABASE IF NOT EXISTS `%s` CHARACTER SET utf8mb4;",
		dbName)
	_, err = db.Exec(createSQL)
	if err != nil{
		panic(err)
	}
}
```

连接数据库

```go
//连接数据库
func ConnectDB(cfg *DBConfig) {
	//初始化为默认配置
	cfg = defaultDbConfig(cfg)
	GlobalConfig = cfg
	//初始化数据库配置
	InitDataBase(cfg)
	dsn := fmt.Sprintf("%s?charset=utf8&parseTime=True&loc=Local",cfg.DSN)

	if db, err := gorm.Open(cfg.DBType, dsn); err != nil{
		panic(err)
	}else{
		db.DB().SetMaxIdleConns(cfg.MaxIdleConns)
		db.DB().SetMaxOpenConns(cfg.MaxOpenConns)
		db.LogMode(cfg.Debug)

		db.BlockGlobalUpdate(true)
		db.SetLogger(new(logger))
		GlobalDB = db
	}
	log.Info("db connected success")
}
```

## 测试

```go
func TestInitDataBase(t *testing.T){
	InitDataBase(&DBConfig{
		DSN:"root:4.234.23123@(127.0.0.1:3306)/somethingWeDontCare",
	})
}

func TestConnectDB(t *testing.T) {
	ConnectDB(&DBConfig{
		DSN:          "root:4.234.23123@(127.0.0.1:3306)/dsp-authorization",
		MaxIdleConns: 10,
		MaxOpenConns: 20,
		AutoMigrate:  false,
		Debug:        false,
		CacheFlag:    true,
	})
}
```

## 使用logger



# 数据操作

## 表模型

> model.go

```go
type Alert struct {
	//CommonModel    //通用结构属性
	Name                   string `json:"name" column:"name"`
	UniqueCode             string `json:"uniqueCode" column:"unique_code"`
	Description            string `json:"description" column:"description"`
	AlertType              string `json:"alertType" column:"alert_type"`
	AlertLevel             string `json:"alertLevel" column:"alert_level"`
	AlertTargetID          string `json:"alertTargetID" column:"alert_target_id"`
	AlertTargetName        string `json:"alertTargetName" column:"alert_target_name"`
	Enable                 string `json:"enable" column:"enable"`
	NotificationStrategyID string `json:"notificationStrategyID" column:"notification_strategy_id"`
	AlertTemplateID        string `json:"alertTemplateID" column:"alert_template_id"`
	ObjectDetailID         string `json:"objectDetailID" column:"object_detail_id"`

}
```

## 编辑配置

进行初始化

```go
func GetConfig() *DBConfig{
	return &DBConfig{
		DSN:          "root:4.234.23123@(127.0.0.1:3306)/dsp-AlertCenter",
		MaxIdleConns: 10,
		MaxOpenConns: 20,
		AutoMigrate:  false,
		Debug:        false,
		CacheFlag:    true,
	}
}
```

## 增删改查等操作

```go
func Migrate() (err error){
	err = GlobalDB.AutoMigrate(&Alert{}).Error
	return
}

// Create 创建数据
func Create(alert *Alert) (err error){
	//create， 增加一条数据
	err = GlobalDB.Create(alert).Error
	return
}

// FirstOrCreate 查询数据，没有则创建
func FirstOrCreate(alert *Alert) (err error){
	//两种写法
	err = GlobalDB.FirstOrCreate(alert, "unique_code = ?", alert.UniqueCode).Error
	//err = GlobalDB.Where("UniqueCode = ?", a.UniqueCode).FirstOrCreate(&a).Error
	return
}

// FindByNameFirst FindFirst 查第一条
func FindByNameFirst(alerts *Alert, name interface{}) (err error){
	//类似的操作还有Take（随机一条）， Last（主键倒排序第一条）
	err = GlobalDB.First(alerts, "name = ?", name).Error
	return
}

// FindByUniqueCode 根据uniqueCode查
func FindByUniqueCode(alert *Alert, uniqueCode interface{}) (err error){
	err = GlobalDB.First(alert, "unique_code = ?", uniqueCode).Error
	//err = GlobalDB.Find(&alert, "UniqueCode = ?", uniqueCode).Error
	return
}

// FindList 查询列表
func FindList() (aLists []*Alert, err error){
	err = GlobalDB.Find(&aLists).Error
	return
}

// Update 更新这条数据
func Update(alert *Alert, value interface{}) (err error) {
	err = GlobalDB.Model(alert).
		Where("unique_code = ?", alert.UniqueCode).
		Update("alert_level", value).
		Error
	return
}

// Delete 删除数据
func Delete(UniqueCode string) (err error){
	err = GlobalDB.Where("unique_code = ?", UniqueCode).Delete(&Alert{}).Error
	return
}

```

## 测试

```go
func init(){
	ConnectDB(GetConfig())
	err := Migrate()
	if err != nil{
		log.Errorf("创建表失败%v",err)
	}
}

func TestCreate(t *testing.T) {
	a := &Alert{Name: "test",
		UniqueCode: "123457",
		Description: "test",
		AlertLevel: "严重"}
	err := Create(a)
	if err != nil{
		t.Errorf("创建失败，err = %v", err)
	}
	t.Log("创建成功")
}

func TestFindByName(t *testing.T) {
	//首先查询出的数据直接初始化成指针类型，应该传参的时候会更快些
	as := &Alert{}
	err := FindByNameFirst(as, "test")
	t.Logf("查询第一条数据为：%v",*as)
	if err != nil{
		t.Errorf("查询记录失败，err : %v", err)
	}
}

func TestFindList(t *testing.T) {
	aList, err := FindList()
	if err != nil{
		t.Errorf("查询列表失败，err : %v", err)
	}
	for _,v := range aList{
		t.Logf("%v\t%v",*v,err)
	}
}

func TestFindByUniqueCode(t *testing.T) {
	a, uniqueCode := &Alert{}, "123457"
	err := FindByUniqueCode(a, uniqueCode)
	if err != nil{
		t.Errorf("使用uniqueCode查询失败，err : %v", err)
	}
	t.Log(*a, uniqueCode)
}

func TestFirstOrCreate(t *testing.T) {
	a := &Alert{Name: "test",
		UniqueCode: "123458",
		Description: "test",
		AlertLevel: "非常严重"}
	err := FirstOrCreate(a)
	if err != nil{
		t.Errorf("插入失败，err : %v", err)
	}
}

func TestUpdate(t *testing.T) {
	a := &Alert{UniqueCode: "123456"}
	err := Update(a, "一般")
	if err != nil{
		t.Errorf("插入失败，err : %v", err)
	}
	fmt.Println(*a)
}

func TestDelete(t *testing.T) {
	err := Delete("123456")
	if err != nil{
		t.Errorf("删除失败，err : %v", err)
	}
}
```

# gormV2

[文档](https://gorm.io/zh_CN/docs/index.html)

## 实践

