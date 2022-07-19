# 请求

## GET

- 设置请求

```go
	//将jaeger中查询到serviceList parse到model中
	client := &http.Client{}
	//访问的url
	url := "http://10.23.12.108:36176/api/services"
	//查询jaeger接口所有的服务
	request, err := http.NewRequest("GET", url, nil)
	if err != nil {
		log.Fatalf("http.NewRequest : %v", err)
	}
	rsp,err := client.Do(request)
	if err != nil{
		log.Fatalf("client.Do : %v", err)
	}
	defer rsp.Body.Close()
```

- 读取rsp.body

```go
    //读取数据
	rspBody, err := ioutil.ReadAll(rsp.Body)
	if err != nil{
		log.Fatalf("ioutil.ReadAll : %v", err)
	}
	fmt.Println(string(rspBody))
```

- 重复读取rep.body

```go
	//有个需求，需要重复读取rsp.Body，但是发现无法重复读取，应该是底层中有偏移指针，重置即可
	//--------重复读取rspBody([]byte)，然后重新生成可读取的(io.ReadCloser)类型---------
	newRspBody:= ioutil.NopCloser(bytes.NewReader(rspBody))
	rspBody, err = ioutil.ReadAll(newRspBody)
```

- parse成结构体

```go
	var services model.Services
	err = json.NewDecoder(bytes.NewReader(rspBody)).Decode(&services)
	if err == io.EOF{
		fmt.Println(services)
	}
	if err != io.EOF&&err != nil{
		log.Fatalf("json.NewDecoder : %v", err)
	}
	fmt.Println(services)
```

