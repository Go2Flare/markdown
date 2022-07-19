# 结构体嵌套

[路径](D:\Project\Test\testStruct)

## 相互嵌套

路口：包含2个路段信息（两个路段连接）

```go
//路口
type intersection struct{
	name string
	section1,section2 *section
}
```

路段：包含一个路口的数组（包含多个路口）

```
//路段
type section struct{
	name string
	intersections []*intersection
}
```

实例：

```go
func main(){
	var A,B,C,D,E,F section
	//路口
	a1,a2,a3,a4,a5 := intersection{"a1",&A,&B},
	intersection{"a2",&A,&C}, intersection{"a3",&A, &D},
	intersection{"a4",&B,&E}, intersection{"a5",&B,&F}
	//路段
	A,B = section{"A", []*intersection{&a1,&a2,&a3}}, section{"B", []*intersection{&a1,&a4,&a5}}

	fmt.Println(a1.section1.intersections[0],a2,a3,a4,a5)
	fmt.Println(A,B)
	//好像能实现两个对象的无限互相调用，也就是说只需要传入一个对象，那么就可以直接操作另一个对象的数据，属性如果都是指针的话，那就都是两边的同一张表操作
	//无限调用对象a1路口,和属性对象A路段,修改A的名字
	a1.section1.intersections[0].section1.intersections[0].section1.intersections[0].section1.intersections[0].section1.name = "modifyA"
	//即使是初始化没有关联的a2路口和B路段，也可以通过互相调用修改对方的属性
	a2.section1.intersections[0].section2.name = "modifyB"
	fmt.Println(*A.intersections[0].section1.intersections[0].section1,B)
}

```

输出：

```go
&{a1 0xc0000c4390 0xc0000c43c0} {a2 0xc0000c4390 0xc0000c43f0} {a3 0xc0000c4390 0xc0000c4420} {a4 0xc0000c43c0 0xc0000c4450} {a5 0xc0000c43c0 0xc0000c4480}
{A [0xc0000a63a0 0xc0000a63c0 0xc0000a63e0]} {B [0xc0000a63a0 0xc0000a6400 0xc0000a6420]}
{modifyA [0xc0000a63a0 0xc0000a63c0 0xc0000a63e0]} {modifyB [0xc0000a63a0 0xc0000a6400 0xc0000a6420]}
```

## 总结

- 互相嵌套意味着两个结构体已经相互紧密联系，特别是在用指针互相嵌套的时候

- 只要有其中一个对象，意味着通过这个对象**调用子属性**对另一个对象进行属性修改操作

# 交叉编译

## windows编译linux

```
set CGO_ENABLED=0
set GOOS=linux
```

```
set CGO_ENABLE=1 GOOS=windows GOARCH=arm64
```



# 函数传参配置

- 输入配置

```go
// NewConfig create a config
func NewConfig(opts ...Option) (*Config, error) {
	var c Config

	// 1. load env config
	envError := envconfig.Process(context.Background(), &c)
	if envError != nil {
		return nil, envError
	}

	// 2. load code config
	for _, opt := range opts {
		opt(&c)
	}

	// 3. merge resource
	parseEnvKeys(&c)
	mergeResource(&c)
	return &c, c.IsValid()
}
```

```go
slsConfig, err := provider.NewConfig(provider.WithServiceName(opts.ServiceName),
		provider.WithServiceVersion(opts.ServiceVersion),
		provider.WithTraceExporterEndpoint(opts.Endpoint),
		provider.WithMetricExporterEndpoint(opts.Endpoint),
		provider.WithSLSConfig(opts.Project, opts.Instance, opts.AccessKey, opts.AccessSecret))
```

- 函数赋值对象

```go
func WithTraceExporterEndpoint(url string) Option {
	return func(c *Config) {
		c.TraceExporterEndpoint = url
	}
}
```

# 实用操作

## 获取当前目录

```go
    //获取当前目录
	curPath, err := os.Getwd()
	//获取当前目录，一般在编译二进制下使用
	dir, err := filepath.Abs(filepath.Dir(os.Args[0]))
	if err != nil {
		log.Fatal(err)
	}
	//创建一个目录
	err = os.Mkdir(filepath.Join(curPath, "newGIFs"), 0666)
	if err != nil {
		err = errors.New("---已进行过修改建立文件，文件如有需要保存请退出将文件转移，否则将被覆盖！---")
		fmt.Println(err)
	}
```

# channel

## 实例

### 循环打印123

```go
package main

import (
	"sync"
)

var count = 100
var wg sync.WaitGroup

func main() {
	//3个协程之间同步的缓冲管道
    						//A   B   C
	chanA := make(chan int, 1)//    <-    (将元素传递chanB)
	chanB := make(chan int, 1)//        <-
	chanC := make(chan int, 1)//<-

	chanA <- 0
	wg.Add(3) //增加3个协程的计数

	go printA(chanA, chanB)
	go printB(chanB, chanC)
	go printC(chanC, chanA)
	wg.Wait()
}

func printA(chanA chan int, chanB chan int) {
	defer wg.Done()

	for i := 0; i < count; i++ {
		//每个协程做管道的传递达到 先后顺序的同步
		<-chanA
		println("1")
		chanB <- 0
	}
}

func printB(chanB chan int, chanC chan int) {
	defer wg.Done()

	for i := 0; i < count; i++ {
		<-chanB
		println("2")
		chanC <- 0
	}
}

func printC(chanC chan int, chanA chan int) {
	defer wg.Done()
	for i := 0; i < count; i++ {
		<-chanC
		println("3")
		chanA <- 0
	}
}
```



# 包管理

- 每个目录下都可以定义为一个包（不能超过）
- 每个包下都可以有一个main函数入口
- 包下的mod文件递归的为所在目录提供dependencies
- 运行完main.go，有如下图的提示表示主go程结束

![image-20220226150846687](http://myimg.go2flare.xyz/img/image-20220226150846687.png)

# 字符串处理

```go
func main(){
	s := "\n                    <p>某日，刘备与赵云谈心：“吾有二弟、三弟，此生足矣。”</p><p>赵云：“末将亦愿誓死效忠将军。”</p><p>刘备：“既如此，你我二人不如就此结拜。”</p><p>赵云：“额.....赵四这个名字不太好吧。”</p>                \n"
	b := []rune(s)
	i,j:=0,len(b)-1
    //首位除中文外的全部去除
	for ;i<len(b)&&(b[i] > 40869 || b[i] < 19968);i++{}
	for ;j>=0&&(b[j] >40869 || b[j] < 19968);j--{}
	if i<=j{
		b = b[i:j+1]
	}
	s = string(b)
	s = strings.ReplaceAll(s, "</p><p>", "\n")
	fmt.Println(s)
}
```

# time包

## 秒，时间单位，符号s(英语：second)：

一秒
健康人的心跳大约持续一秒。美国人平均每一秒吃掉350块比萨饼。地球每一秒绕太阳旋转30公里，而与此同时太阳在银河系中穿行274公里。一秒钟不足以使月光到达地球（需1.3秒）。传统意义上，一秒是24分之一天的60分之一的60分之一，但是科学家给出了一个更精确的定义：铯133原子基态超精细能阶跃迁的9 192 631 770个周期所持续的时间，称为一秒

0.000 000 001 毫秒 = 1皮秒

0.000 001 毫秒 = 1纳秒

0.001 毫秒 = 1微秒

1毫秒=0.001秒

60秒=1分钟

60分钟=1小时

24小时=1天

7天=1星期

30，31，28或29天=1月

12月=1年

100年=1世纪。

## 毫秒，时间单位，符号ms（英语：millisecond ）：1秒的千分之一（10-3秒）

一毫秒（千分之一秒）
典型照相机的最短曝光时间为一毫秒。一只家蝇每三毫秒扇一次翅膀；蜜蜂则每五毫秒扇一次。由于月亮绕地球的轨道逐渐变宽，它绕一圈所需的时间每年长两毫秒。在计算机科学中，10毫秒的间隔称为一个jiffy。

0.000 000 001 毫秒 = 1皮秒
0.000 001 毫秒 = 1纳秒
0.001 毫秒 = 1微秒
1000 毫秒 = 1秒

## 微秒，时间单位，符号μs（英语：microsecond ）：1秒的百万分之一（10-6秒）

一微秒（百万分之一秒）
光在这个时间里可以传播300米，大约是3个足球场的长度，但是海平面上的声波只能传播1/3毫米。高速的商业频闪仪闪烁一次大约持续1微秒。一筒炸药在它的引信烧完之后大约24微秒开始爆炸。

0.000 001 微秒 = 1皮秒
0.001 微秒 = 1纳秒
1,000 微秒 = 1毫秒
1,000,000 微秒 = 1秒

## 纳秒，时间单位，符号ns（英语nanosecond）：

一纳秒（十亿分之一秒）
光在真空中一纳秒仅传播30厘米（不足一个步长）。个人电脑的微处理器执行一道指令（如将两数相加）约需2至4纳秒。另一种罕见的亚原子粒子K介子的存在时间为12纳秒。

1秒的10亿分之一。常用作内存读写速度的单位，其前面数字越小表示速度越快。



## 时间戳与time格式互相转换

```go
func main() {
    //Time类型.Unix  是将Time类型转为时间戳
    timestamp := time.Now().Unix()//time.Now()是当前时间（Time类型）
    fmt.Println("now",timestamp)

    //time.Unix  是time包里的函数，将时间戳转为Time类型
    fmt.Println(time.Unix(timestamp, 0))
}

//输出：
//now 1550377621
//2019-02-17 12:27:01 +0800 CST
```

