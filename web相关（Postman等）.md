# web相关（Postman等）

接口查看法，是我们最常用的定位前后端问题的方法。即：一般用来查看是后端返回给前端的数据有误，还是前端显示有误。
主流浏览器（如Chrome，FireFox，等）都有自带的接口查看工具，可以通过F12（设置–工具–开发者工具）开启抓包。每进行一个操作，一般都会调用对应的接口，在NetWork中可以看到当前页面发送的每个请求。以谷歌浏览器为例：

1、进入 NetWork页面
如图，按F12，切换到NetWork页面，默认展示的是All页面。.js、 .css 、.ico、.png 这些结尾的都是前端的渲染、图标、图片等，不是接口。![在这里插入图片描述](https://img-blog.csdnimg.cn/a4032fa8cc994289a30bfe548c4a4911.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6Iqx5rWL6K-V,size_20,color_FFFFFF,t_70,g_se,x_16)

2、点击Fetch/XHR，这里可以看到页面发起的接口![在这里插入图片描述](https://img-blog.csdnimg.cn/2b3b008d9621483ea6c63d095686bdf2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6Iqx5rWL6K-V,size_20,color_FFFFFF,t_70,g_se,x_16)

3、找到出问题的接口
很多时候，执行一个操作，可能调用了多个接口，就需要判断当前问题对应的是哪个接口：
1）查看开发的接口文档；
2）如果没有接口文档，清空Network列表，进入测试界面。然后执行操作，如果有多个请求，逐个查看哪个接口的入参或响应数据是你刚操作的数据；
3）如果没有接口文档，出入参还是加密的，我也不会（我只会逐个接口解密入参看是哪个，或者直接问开发）；
4）最便捷的：问你的开发，是哪个接口（如果有文档还一直问，怕是会被骂）。

4、NetWork页面怎么看接口详情
一般响应数据在 preview 页面看比较清晰，内容和 Response页面是一样的。![在这里插入图片描述](https://img-blog.csdnimg.cn/52ad84a59c3e4c9ea5db6e17e4c87c0b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6Iqx5rWL6K-V,size_20,color_FFFFFF,t_70,g_se,x_16)

5、问题定位
1）一般情况下，console中的报错信息属于前端问题。但是，有些时候console有报错，但不是bug（看是否影响功能）；
2）请求地址错误、请求体参数缺漏或错误，这些都是前端问题；
3）response 或 preview 中，没有返回值，或者返回的内容错误（与操作结果不符，有权限的话可以查看数据库），是后端问题。
4）前端界面显示的内容，与后端接口返回的内容不一样，是前端问题。

# HTTP请求

```go
func HttpGet(){
	client := &http.Client{}
	//生成要访问的url
	url := "http://somesite/somepath/"
	//提交请求
	reqest, err := http.NewRequest("GET", url, nil)

	//增加header选项
	reqest.Header.Add("Cookie", "xxxxxx")
	reqest.Header.Add("User-Agent", "xxx")
	reqest.Header.Add("X-Requested-With", "xxxx")

	if err != nil {
		panic(err)
	}
	//处理返回结果
	response, _ := client.Do(reqest)
	defer response.Body.Close()
}

func HttpPost() {
	//Golang发送post请求　　
	post := `{"待发送":"json"}`
	fmt.Println(post)
	api_url := ""
	var jsonstr = []byte(post) //转换二进制
	buffer := bytes.NewBuffer(jsonstr)
	request, err := http.NewRequest("POST", api_url, buffer)
	if err != nil {
		fmt.Printf("http.NewRequest%v", err)
	}
	request.Header.Set("Content-Type", "application/json;charset=UTF-8") //添加请求头
	client := http.Client{}                                              //创建客户端
	resp, err := client.Do(request.WithContext(context.TODO()))          //发送请求
	if err != nil {
		fmt.Printf("client.Do%v", err)
	}
	respBytes, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("ioutil.ReadAll%v", err)
	}
	fmt.Println(respBytes)
}
func main(){
	HttpGet()
	HttpPost()
}
```

# HTTP2.0

## 查看网站证书

f12 -> 安全 -> 复制到文件

# uuid

```go
package main

import (
   "fmt"
   uid "github.com/satori/go.uuid"
   "reflect"
)

func main(){
   u1 := uid.Must(uid.NewV4(), nil).String()
   fmt.Println(u1, reflect.TypeOf(uid.Must(uid.NewV4(), nil)))
}
```

# template

- 服务端渲染

Golang为模板操作提供了丰富的支持，嵌套模板、导入函数、表示变量、迭代数据等都很简单。若需要比CSV数据格式更复杂的电脑关系，模板可能是一个不错的解决方案。模板的另一个 应用是网站的页面渲染。

Golang内置`text/template`和`html/template`两个模板库，`html/template`库为HTML提供了完整的支持，包括普通变量渲染、列表渲染、对象渲染等。

`text/template`是Golang标准库，实现数据驱动模板以生成文本输出，可理解为一组文本按照特定格式嵌套动态嵌入另一组文本中。可用来开发代码生成器。

`text/template`包是数据驱动的文本输出模板，即在编写好的模板中填充数据。一般而言，模板使用流程分为三个步骤：定义模板、解析模板、数据驱动模板。



```shell
$ vim main.go
```



```go
package main

import (
    "os"
    "text/template"
)

func main() {
    //创建模板
    id := 100
    tmpl, err := template.New("tplname").Parse("id = {{.}}")
    if err != nil {
        panic(err)
    }
    //模板合并输出
    err = tmpl.Execute(os.Stdout, id)
    if err != nil {
        panic(err)
    }
}
```



```shell
$ go run main.go
id = 100
```

## Action

- `{{.}}`中`.`点号表示传入模板的数据，数据不同渲染不同。
- `.`点可代表Golang的任何类型，比如Struct、Map等。
- `{{`和`}}`包裹的内容统称为`Actions`

模板的输入文本是任何格式的UTF-8编码文本，，`action`可分为两种：

- 数据求值（data evaluations）
   `action`数据求值的结果会直接复制到模板中
- 控制结构（control structures）
   `action`控制结构和Golang程序类似，也是条件语句、循环语句、变量、函数调用等...

将模板成功解析后可安全地在并发环境中使用，若输出到同一个`io.Writer`数据可能会重叠，不能保证并发执行的先后顺序。

## template.New

```go
func New(name string) *Template
```

`template.New()`创建名为`name`的模板

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    //模板变量
    type User struct {
        Id   int
        Name string
    }
    user := User{100, "admin"}
    //创建模板
    tplname := "user"
    tmpl := template.New(tplname)
    //解析模板
    text := "id = {{.Id}} name = {{.Name}}"
    tpl, err := tmpl.Parse(text)
    if err != nil {
        panic(err)
    }
    //渲染输出
    err = tpl.Execute(os.Stdout, user)
    if err != nil {
        panic(err)
    }
}
```



```shell
$ go run main.go
id = 100 name = admin
```

## template.Parse

```go
func (t *Template) Parse(text string) (*Template, error)
```

`template.Parse()`方法将字符串`text`解析为模板，嵌套定义的模板会关联到最顶层`t`。

`template.Parse()`可以多次调用，但只有第一次调用可以包含空格、注释、模板定义之外的文本。若后续调用解析后仍剩余文本会引发错误、返回`nil`、丢弃剩余文本。若解析得到的模板已有相关联的同名模板则会覆盖原模板。

## template.ParseFiles

`ParseFiles`方法接收 一个字符串，字符串的内容是一个模板文件的路径，可为绝对路径或相对路径。

解析模板文件

```shell
$ vim view/default.html
```



```html
id = {{.Id}} name = {{.Name}}
```

解析模板文件

```shell
$ vim main.go
```



```go
package main

import (
    "os"
    "text/template"
)

func main() {
    //模板变量
    type User struct {
        Id   int
        Name string
    }
    user := User{100, "admin"}
    //创建模板
    tmpl, err := template.ParseFiles("./view/default.html")
    if err != nil {
        panic(err)
    }
    //模板合并输出
    err = tmpl.Execute(os.Stdout, user)
    if err != nil {
        panic(err)
    }
}
```



```shell
$ go run main.go
id = 100 name = admin
```

## template.ExecuteTemplate

多模板时指定模板

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    //模板变量
    type User struct {
        Id   int
        Name string
    }
    user := User{100, "admin"}
    //创建模板
    tplname := "tplname"
    tmpl := template.New(tplname)
    //解析模板
    text := "id = {{.Id}} name = {{.Name}}"
    tpl, err := tmpl.Parse(text)
    if err != nil {
        panic(err)
    }
    //渲染输出
    err = tpl.ExecuteTemplate(os.Stdout, tplname, user)
    if err != nil {
        panic(err)
    }
}
```

## template.Execute

`template.Execute()`方法将解析好的模板应用到`data`，并将输出写入`wr`。若执行时出现错误则会停止执行，但 有可能已经写入部分数据，模板可以安全的并发执行。

```go
func (t *Template) Execute(wr io.Writer, data interface{}) (err error)
```

输出到指定文件

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    //模板变量
    type User struct {
        Id   int
        Name string
    }
    user := User{100, "admin"}
    //创建模板
    tplname := "tplname"
    tmpl := template.New(tplname)
    //解析模板
    text := "id = {{.Id}} name = {{.Name}}"
    tpl, err := tmpl.Parse(text)
    if err != nil {
        panic(err)
    }
    //输出文件
    file, err := os.OpenFile("./public/tpl.txt", os.O_CREATE|os.O_WRONLY, 0755)
    if err != nil {
        panic(err)
    }
    //渲染输出
    err = tpl.Execute(file, user)
    if err != nil {
        panic(err)
    }
}
```

读取模板文件解析后输出到指定位置

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    //模板变量
    type User struct {
        Id   int
        Name string
    }
    user := User{100, "admin"}
    //解析模板
    tplFile := "./view/default.html"
    tpl, err := template.ParseFiles(tplFile)
    if err != nil {
        panic(err)
    }
    //输出文件
    outFile := "./runtime/view/default.html"
    file, err := os.OpenFile(outFile, os.O_CREATE|os.O_WRONLY, 0755)
    if err != nil {
        panic(err)
    }
    //渲染输出
    err = tpl.Execute(file, user)
    if err != nil {
        panic(err)
    }
}
```

# 测试环境

## 添加header鉴权元素

![image-20220217102801899](http://r60pkrx4f.hn-bkt.clouddn.com/img/image-20220217102801899.png)

## 增加pre-request Script自动添加header

![image-20220217102918246](http://r60pkrx4f.hn-bkt.clouddn.com/img/image-20220217102918246.png)

### 实例

在Pre-request Script 中增加代码 可以实现在请求时自动添加Header ,可在Collection中添加

```javascript
// 添加新 header
pm.request.headers.add({
    key: 'Accept-Encoding',
    value: 'gzip'
});

// 添加或修改已存在 header
pm.request.headers.upsert({
    key: 'Connection',
    value: 'close'
});

// 移除 header
pm.request.headers.remove('User-Agent')
```

使用在get: metrics中

```js
pm.request.headers.add({
    key: 'username',
    value: '{{username}}'
})

pm.request.headers.add({
    key: 'Authorization',    
    value: '{{Authorization}}'
})
```





# 接口测试相关

## Postman

- 请求页签

​			**Params**：get请求传参

​			**authorization**：鉴权

​			**headers**：请求头

​			**Body**：post请求传参

​					**form-data**：既可以键值传参也可以传输文件

​					**x-www-form-urlencoded**：只能够传键值对参数

​					**raw**：text， json，xml，html，javaScript

​					**binary**：把文件以二进制的形式传输

​			**pre-request-script**：请求之前的脚本

​			**tests**：请求之后的断言

​			**cookies**：用于管理cookie信息

- 响应页签

  ​	**Body**：接口返回的数据

  ​			**Pretty**：以Json，XML，html...不同的格式查看返回的数据

  ​			**Raw**：以文本的方式查看返回的数

  ​			**PreView**：以网页的方式查看返回的数据

  ​	**Cookies**：响应的Cookie信息

  ​	**Hesaders**：响应头

  ​	**Test Result**：断言的结果

  ​	200（状态码）

  ​	OK（状态信息）

  ​	681ms（响应的时间）

  ​	343B（响应的字节数）

- 面试题

  GET和POST的区别？

  1. get请求一般是获取数据，post请求一般的提交数据

  2. post请求和get请求安全

  3. 本质是传参的方式不一样：

     get请求在地址栏后面以？的形式传参，多个参数间以&相隔

     post请求本质上是在body以表单形式传输

## 全局变量环境变量

- 环境变量：该环境范围内使用的全局变量
- 全局变量：全局变量是能够在任何接口里面访问的变量。
- 获取环境变量和全局变量的值通过:**{{变量名}}**

> host : https://api.weixin.qq.com

## 接口关联

### 第一个接口access_token保存变量

1. 使用json提取器

   ```javascript
   // 获取响应body
   console.log(responseBody);
   // 使用json提取器提取access_token
   var repJSON = JSON.parse(responseBody); //字符串形式转对象形式
   console.log(repJSON.access_token, repJSON);
   // 将access_token设置为全局变量/环境变量
   pm.environment.set("access_token", repJSON.access_token);
   ```

2. 使用正则提取器

   ```js
   // 使用正则提取器实现接口关联, match匹配
   var repJSON = responseBody.match(new RegExp('"access_token":"(.*?)"'))[1];
   pm.environment.set("access_token", repJSON);
   console.log(repJSON);
   ```

### 第二个接口使用

![image-20220217144922483](http://r60pkrx4f.hn-bkt.clouddn.com/img/image-20220217144922483.png)

## 内置动态参数与自定义动态参数

1. postman内置动态参数

   {{$timestamp}}	生成当前时间戳

   {{$randomint}}	生成0-1000之间的随机数

   {{$guid}}				生成速记GUID字符串

2. 自定义动态参数

   ```js
   var time = Date.now();//手动获取时间戳
   // 设置环境变量
   pm.environment.set("time", time);
   ```

## 创建和修改接口关联

创建接口：

> POST {{host}}/cgi-bin/tags/create?access_token={{access_token}}

1. pre-request设置动态变量

   ```js
   var time = Date.now();//手动获取时间戳
   // 设置环境变量
   pm.environment.set("time", time);
   ```

2. test设置获取的reponse.body中的tag.id保存为变量(文件夹)

   ```js
   //正则获取创建的标签ID
   var tag_id = responseBody.match(new RegExp('"id":(.*?),'))[1];
   console.log(tag_id);
   // 标签设置为文件夹变量
   pm.collectionVariables.set("tag_id", tag_id);
   ```

修改接口：

> POST {{host}}/cgi-bin/tags/update?access_token={{access_token}}

1. post.body 使用json格式传输

   ```json
   {
       "tag": {
           "id": "{{tag_id}}",
           "name": "flare{{$timestamp}}"
       }
   }
   ```

删除接口：

> POST {{host}}/cgi-bin/tags/delete?access_token={{access_token}}

request.body带上

```js
{"tag":{"id":{{tag_id}}}}
```

## 文件上传接口

> POST  {{host}}/cgi-bin/tags/create?access_token={{access_token}}

- 传输body选择form-data

  可以**传输文本或者文件**

- response

  ```json
  {
      "url": "http://mmbiz.qpic.cn/mmbiz_png/TH9uYG1HxGG33eWbonWKxSER60J2icRJjJkx8GE7FUBqaFaayP6AHZiaXDK1tiboeeQiclRVTZuegQcAQeVp2krj2w/0"
  }
  ```

## postman断言

在编辑请求界面中的Tests

![image-20220217155557003](http://r60pkrx4f.hn-bkt.clouddn.com/img/image-20220217155557003.png)

## 官方demo

**Status code : Code is 200	检查返回的状态码是否为200**

**Response body : Contains string	检查响应中包括指定字符串**

**Response body : JSON value check	检查响应中其中JSON的值**

**Response body : is equal to a string	检查响应等于一个字符串**

Response headers : Content-Type	检查是否包含响应头Content-Type

Response time is less than 200ms	检查请求耗时小于200ms

### 断言中获取自定义动态参数（环境变量）的方式

- pm.environment.get("time")
- environment["time"]
- environment.time

### 全局断言

> 测试状态码之类的

## 批量运行

![image-20220217164642442](http://r60pkrx4f.hn-bkt.clouddn.com/img/image-20220217164642442.png)

## 实现csv，json数据驱动

定义测试数据

### csv

```csv
grant_type,appid,secret,assert_value
client_credential,wx7bef920005035d1a,a77cd6deb5dd5d1722bc0f5494997f89,access_token
,wx7bef920005035d1a,a77cd6deb5dd5d1722bc0f5494997f89,40002
client_credential,,a77cd6deb5dd5d1722bc0f5494997f89,41002
client_credential,wx7bef920005035d1a,,41004
```

### json

```json
[
	{"grant_type":"client_credential","appid":"wx7bef920005035d1a","secret":"a77cd6deb5dd5d1722bc0f5494997f89","assert_value":"access_token"},
	{"grant_type":"","appid":"wx7bef920005035d1a","secret":"a77cd6deb5dd5d1722bc0f5494997f89","assert_value":40002},
	{"grant_type":"client_credential","appid":"","secret":"a77cd6deb5dd5d1722bc0f5494997f89","assert_value":41002},
	{"grant_type":"client_credential","appid":"wx7bef920005035d1a","secret":"","assert_value":41004}
]
```

### 更改请求

![image-20220217172941944](http://r60pkrx4f.hn-bkt.clouddn.com/img/image-20220217172941944.png)

### 断言修改

```js
// 判断当返回结果中包括由access_token时才通过正则表达式取值
if (responseBody.search("access_token")!=-1){
    // 使用正则提取器实现接口关联, match匹配
    var repJSON = responseBody.match(new RegExp('"access_token":"(.*?)"'))[1];
    pm.environment.set("access_token", repJSON);
    console.log(repJSON);
}
// 状态断言
pm.test("检查返回的状态码为200", function () {
    pm.response.to.have.status(200);
});

// 业务断言
if (data.status != undefined){
    pm.test("Body matches string", function () {
        pm.expect(pm.response.text()).to.include(data.assert_value);
    });
}
```

## 常见的请求头headers

**Accept	客户端的用户类型**

**X-Requested-With	异步请求**

**User-Agent	客户端的用户类型**

**Cookie	cookie信息**

**Content-Type	请求内容的格式**

Referer	来源

Connection	连接方式

Host	请求的主机地址

## Postman接口MockServer服务器

后端接口尚未开发完成，前端业务需要调用后端的接口

![image-20220217174843757](http://r60pkrx4f.hn-bkt.clouddn.com/img/image-20220217174843757.png)

https://d42c0fe5-6c8c-4109-bc5e-3d690aba12cb.mock.pstmn.io

## default里设置

![image-20220217175354257](http://r60pkrx4f.hn-bkt.clouddn.com/img/image-20220217175354257.png)

## 测试地址

https://d42c0fe5-6c8c-4109-bc5e-3d690aba12cb.mock.pstmn.io/test

```json
{"error_code":404, "msg":"返回成功", "data":[]}
```



# cookie

## what is cookie?

cookie是服务器产生的保存在客户端的一小段文本信息，格式是字典，键值对

cookie有两类：

1. 会话级cookie：保存在内存，当浏览器关闭会自动消失
2. 持久化cookie：保存在硬盘，浏览器关闭不会消失，生命周期取决于失效时间



## cookie鉴权实现

第一步：客户端第一次访问服务器时生成cookie，然后通过响应头中的Set-cookie传输到客户端，然后再客户端保存

第二步：客户端第2-n次访问服务器时，那么在请求头cookie里面就会自动的带上客户端保存的cookie，然后和服务器的cookie进行对比来实现鉴权



Postman：自动管理

Apache Jmeter:通过http cookie管理器自动管理



## cookie缺点

服务器生成保存在客户端，对于一些重要信息，用户名，密码通过cookie保存不安全

把敏感的数据保存到服务器，把不敏感的数据保存到cookie

# session及鉴权原理

1. session实现鉴权

   第一步：当客户端登录服务器的时候，服务器生成**sessionid**并且保存到服务器，然后再登录请求的响应头里面就会把**sessionid**通过cookie传输给客户端

   第二步：后面所有的请求都在请求头cookie上带上sessionid，然后和服务器的sessionid进行比对实现鉴权

2. 生命周期

   默认时半小时，每半小时登录一下的sessionid不一样

   设置：

   ```
   tomcat/conf/web.xml
   ```

   测试：

   登录失效后看系统的反应，session()的功能测试，修改失效时间

   接口自动化测试：requests.session()

服务器集群，负载均衡的需求逐渐无法满足

# token以及token鉴权的原理和实战

token:令牌，鉴权码

1.token如何实现鉴权？

第一步：一般是登录之后自动生成token或者通过一个单独的接口来生成token，保存在服务器的硬盘，一般保存在服务器的数据库

第二步：后面的所有的请求都必须带上token（请求头，参数），然后和服务器的token对比实现鉴权

## 加密

 对称加密：DES，AES			可以解密

双钥加密：AES						可以解密

MD5，SHA1							不可解密

金融项目，银行项目，第三方支付：自定义加密规则，sign签名

## 应用

功能测试：

​		登录失效后看系统反应，session的功能测试

接口测试：

​		postman：自动管理

​		jmeter：http cookie管理器

自动化测试：

​		接口自动化：token鉴权，接口关联

​		接口自动化：request.get(cookie=cookie)

​		接口自动化：request.session()

​		Web自动化：通过cookie实现万能验证码

## cookie，session，token相同点不同点

**相同点**：都是用于鉴权，由服务器产生

**不同点**：

1. cookie储存在客户端，session储存在服务器内存，token储存在服务器硬盘，session和token的安全性比cookie高
2. token的优势是比session更省资源，不需要管理sessionid



# Newman

postman专门为接口测试，而Newman专为postman而生，newman可以让我们的postman脚本以非gui（脚本）运行

命令： newman run

常用参数：

​		-e	引用环境变量

​		-g	引用全局变量

​		-d	引用数据文件

​		-n	指定测试用例迭代次数

​		-r 	cli,html,json,junit --reporter-html-export report.html



执行命令如下：

```
newman run collection.json -e environmen.json -g globals.json -d data.json -r cli,html,json,junit --reporter-html-export report.html
```

# Postman+Newman+Jenkins接口测试持续集成

1.新建一个项目
2.设置目定义工作空间。
3.执行windows的批处理命令

4.执行系统的Groovy脚本

5.生成的HTMl的报告集成到Jenkins





# 正则

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129111027285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ2MzY1NTM0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129111213473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ2MzY1NTM0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129111213473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ2MzY1NTM0,size_16,color_FFFFFF,t_70)





# APIFOX

1. 背景
作为互联网行业技术从业者，接口调试是必不可少的一项技能，通常我们都会选择使用 Postman 这类工具来进行接口调试，在接口调试方面 Postman 做的确实非常出色。当然除了Postman，还有它的老婆: Postwoman也同样很出色，公号之前有发表过一篇关于它老婆Postwoman使用的文章，详细可阅：推荐一款 Postman 的开源替代品：Postwoman

但是在整个软件开发过程中，接口调试只是其中的一部分，还有很多事情 Postman 是无法完成的，或者无法高效完成，比如：接口文档定义、Mock 数据、接口自动化测试等等。

今天给大家推荐的一款神器: Apifox，它是集：接口文档管理、接口调试、Mock、接口自动化测试于一体的。有的读者咋一看，会觉得和YAPI有点类似，但两者在功能特色上，只能说是各有千秋的。

细心的读者，会发现文章，读到这里，本文就已经为大家推荐了三款效率神器: Apifox、Postwoman、YAPI。

2. 聊一聊接口管理的现状
对于接口管理的现状来说，目前行业大部分采取的解决方案有如下几种：

使用 Swagger 管理接口文档。

使用 Postman 调试接口。

使用 RAP或Easy Mock来进行 Mock 数据。

使用 JMeter 做接口自动化测试。

而上述的接口管理手段，咋一看，貌似没有什么问题，但仔细分析，不难发现，当中存在的问题还真不少，比如要维护不同工具，并且这些工具之间数据一致性非常困难、非常低效。这里不仅仅是工作量的问题，更大的问题是多个系统之间数据不一致，导致协作低效，频繁出问题，开发人员、测试人员工作起来也痛苦不堪。

设想一下这样的一个协作流程（官方示例）：

开发人员在 Swagger 定义好文档后，接口调试的时候还需要去 Postman 再定义一遍。

前端开发 Mock 数据的时候又要去 RAP 或Easy Mock定义一遍，手动设置好 Mock 规则。

测试人员需要去 JMeter 定义一遍。

前端根据  RAP 或Easy Mock定义 Mock 出来的数据开发完，后端根据 Swagger 定义的接口文档开发完，各自测试测试通过了，本以为可以马上上线，结果一对接发现各种问题：原来开发过程中接口变更，只修改了 Swagger，但是没有及时同步修改 RAP 或Easy Mock。

同样，测试在 JMeter 写好的测试用例，真正运行的时候也会发现各种不一致。

时间久了，各种不一致会越来越严重。

3. Apifor介绍
官方对Apifor定位是，Apifox = Postman + Swagger + Mock + JMeter，如下图所示


Apifox目标是通过一套系统、一份数据，解决多个系统之间的数据同步问题。只要定义好接口文档，接口调试、数据 Mock、接口测试就可以直接使用，无需再次定义；接口文档和接口开发调试使用同一个工具，接口调试完成后即可保证和接口文档定义完全一致。高效、及时、准确！

官方地址：https://www.apifox.cn/#

概括来讲，Apifox常用功能分为四类：

接口文档定义功能：Apifox 遵循 OpenApi 3.0 (原Swagger)、JSON Schema 规范的同时，提供了非常好用的可视化文档管理功能，零学习成本，非常高效。

接口调试功能：Postman 有的功能，比如环境变量、预执行脚本、后执行脚本、Cookie/Session 全局共享 等功能，Apifox 都有，并且和 Postman 一样高效好用。

数据 Mock功能：内置 Mock.js 规则引擎，非常方便 mock 出各种数据，并且可以在定义数据结构的同时写好 mock 规则。支持添加“期望”，根据请求参数返回不同 mock 数据。最重要的是 Apifox 零配置 即可 Mock 出非常人性化的数据，具体在本文后面介绍。

接口自动化测试：提供接口集合测试，可以通过选择接口（或接口用例）快速创建测试集。目前接口自动化测试更多功能还在开发中！目标是：JMeter 有的功能基本都会有，并且要更好用。


4. Apifor小试牛刀
接下来，带着大家，简单体验一下Apifor的使用。

1、先在官网下载对应系统安装包，进行安装，安装完成后，第一次启动需要先登录。


Ps:  登录前，需要先通过邮箱来注册一个帐号。

2､ 登录成功后，Apifox默认给了一些例子，单纯看它的界面会发现和Postman界面比较相似。


Ps: 当然也不要被它的外表所欺骗，功能还是有别于Postman的。

3､在本地启一个API服务，端口为8000, 在Apifor上，新建一个新的测试环境，如下所示：


4､新建一个测试分类如：接口测试，也可直接在默认分类上，新建一条接口用例，如下所示：


如上图，添加对应的基础信息、配置请求参数等。

5､选择测试环境，点击发送按钮，运行接口测试用例。


看到这里，可能有些读者觉得和Postman功能基本是一样的，不妨接着往下看。

5. Apifor更多特性
1、调试时自动校验数据结构
使用 Apifox 调试接口的时候，系统会根据接口文档里的定义，自动校验返回的数据结构是否正确，无需通过肉识别，也无需手动写断言脚本检测，非常高效！


根据官方的示例可以看出，在运行集合测试时，可以结合自动校验数据结构的功能， 清晰展示出失败用例校验不通过的原因。



2､零配置 Mock 出非常人性化的数据
1､ 为上述示例，添加一个mock测试服务，配置如下所示：


按照接口字段数据格式要求，根据mock.js语法，配置保存完毕，运行后，自动生成一个mock服务。


其中，Mock.js语法示例可见：http://mockjs.com/examples.html

可以看出 Apifox 零配置 Mock 出来的数据和真实情况是非常接近的，前端开发可以直接使用，而无需再手动写mock规则。

3､代码自动生成
根据接口模型定义，自动生成各种语言/框架（如 TypeScript、Java、Go、Swift、ObjectiveC、Kotlin、Dart、C++、C#、Rust 等）的业务代码（如 Model、Controller、单元测试代码等）和接口请求代码。目前 Apifox 支持 130 种语言及框架的代码自动生成。


img
更重要的是：你可以通过自定义代码模板来生成符合自己团队的架构规范的代码，满足各种个性化的需求。

4､导入、导出
支持导出 OpenApi (原Swagger)、Markdown、Html 等数据格式，因为可以导出OpenApi格式数据，所以你可以利用 OpenApi (Swagger) 丰富的生态工具完成各种接口相关的事情。

支持导入 OpenApi (原Swagger)、Postman、HAR、RAP2、yapi、Eolinker、DOClever、ApiPost 、Apizza 等数据格，方便迁移旧项目。



6. 小结
虽然Apifox目前有些功能还并不完善，但整的来说，Apifox还是不错的，也为接口开发调试测试提供了一种效率更佳的的解决方案，按照Apifox开发团队后续规划，后续会重加增加接口性能测试能力支持（类似JMeter）、支持离线团队多人协作等特性。
