```
microClient.Cal err: {"id":"go.micro.client","code":500,"detail":"service go.micro.srv.getCaptcha: not found","status":"Internal Server Error"}
```

失败配置记录

```go
func main() {
	//初始化consul注册服务
	regOpt := registry.Option(func(options *registry.Options){
		//这里连接localhost居然可以用？
		options.Addrs= []string{"localhost:8500"}
	})
	consulRegistry := consul.NewRegistry(regOpt)

	//初始化reds
	model.InitRedis()

	// New Service
	service := micro.NewService(
		//指定微服务连接服务发现的端口
        //容器中监听localhost网卡是没用的！！！
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

成功配置

```go
func main() {
	//初始化consul注册服务
	regOpt := registry.Option(func(options *registry.Options){
		options.Addrs= []string{"172.17.0.1:8500"}
	})
	consulRegistry := consul.NewRegistry(regOpt)

	//初始化reds
	model.InitRedis()

	// New Service
	service := micro.NewService(
		//指定微服务连接服务发现的端口
		micro.Address(":52666"), //防止随机生成port
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

```
2022/03/17 18:32:09 [Recovery] 2022/03/17 - 18:32:09 panic recovered:
runtime error: invalid memory address or nil pointer dereference
/usr/local/go/src/runtime/panic.go:212 (0x44b459)
/usr/local/go/src/runtime/signal_unix.go:717 (0x44b2a8)
/web/controller/user.go:70 (0x1032439)
        GetImageCd: json.Unmarshal(resp.Img, &img)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.4/context.go:165 (0x9b11ba)
/go/pkg/mod/github.com/gin-contrib/sessions@v0.0.3/sessions.go:52 (0x9c85bc)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.4/context.go:165 (0x9b11ba)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.4/recovery.go:99 (0x9c6158)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.4/context.go:165 (0x9b11ba)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.4/logger.go:241 (0x9c52c0)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.4/context.go:165 (0x9b11ba)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.4/gin.go:489 (0x9bbbbc)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.4/gin.go:445 (0x9bb32b)
/usr/local/go/src/net/http/server.go:2836 (0x6f44b2)
/usr/local/go/src/net/http/server.go:1924 (0x6efd6b)
/usr/local/go/src/runtime/asm_amd64.s:1373 (0x4655f0)
```

