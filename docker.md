# [安装docker](https://docs.docker.com/engine/install/centos/)

## 配置国内源

## 目录

```plain
/etc/docker/daemon.json
```

### 配置

```plain
{
  "registry-mirrors": [
    "https://xx4bwyg2.mirror.aliyuncs.com",
    "http://f1361db2.m.daocloud.io",
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

### desktop配置

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650686420316-4559dc8a-dc57-41af-996a-cafb4ae77f2e.png)

```json
  "insecure-registries": [
    "10.29.3.30:30004"
  ],
  "registry-mirrors": [
    "http://mirrors.aliyun.com",
    "https://mirrors.tuna.tsinghua.edu.cn"
  ]
```

### 设置开机自启

```plain
systemctl enable docker.service
```

## 配置proxy

```plain
"registry-mirrors": [
    "http://mirrors.aliyun.com",
    "https://mirrors.tuna.tsinghua.edu.cn"
  ],
```

## docker desktop设置proxy

### docker仪表盘

​	看日志
​	用容器终端
​	组织容器的生命周期



## 容器 container

​	容器是一个与主机其他进程相隔离的进程使用的内核空间，可以长时间运行

### 操作

- 查看容器

```shell
docker ps -a
```

- docker容器重命名

```plain
容器重命名语法： docker rename 旧容器名 新容器名
```

- 停止容器

```shell
docker stop 容器id
```

- 停止所有容器

```shell
docker stop $(docker ps -q)
```

- 删除容器

```shell
docker rm 容器id
# 强制删除
docker rm -f 容器id/容器名
```

- 删除所有容器

```shell
docker rm $(docker ps -q)
```

-  不进入容器
  查看信息 

```shell
获取容器的hostname：docker exec tomcat001 hostname
获取容器ip地址：
docker exec tomcat001 ip addr
docker inspect 容器ID | grep IPAddress
获取容器环境变量：docker exec tomcat001 env
```

```shell
查看docker name：

sudo docker inspect -f='{{.Name}}' $(sudo docker ps -a -q)

查看dockers ip：

sudo docker inspect -f='{{.NetworkSettings.IPAddress}}' $(sudo docker ps -a -q)

综上，我们可以写出以下脚本列出所有容器对应的名称，端口，及ip

 docker inspect -f='{{.Name}} {{.NetworkSettings.IPAddress}} {{.HostConfig.PortBindings}}' $(docker ps -aq)
```

- 查看日志


```shshellell
docker logs 容器名/容器id
```

- 进入容器

```shell
docker exec -it 容器名or容器id /bin/bash
docker exec -it yzf-web /bin/sh
docker inspect 容器名
```

- 退出容器

```shell
ctrl+p+q
或者直接exit
```

- 主机文件与容器文件交互

```shell
docker cp tracker:/etc/fdfs/client.conf /diskdata/fdfs/ #容器文件fu'zhi
docker cp /diskdata/fdfs/client.conf tracker:/etc/fdfs
```

- 运行容器

```shell
-d 后台运行容器
-p 绑定端口
-t 分配伪tty（容器id）
--name storage 命名容器
```

```shell
//宿主机127.0.0.1:5001->容器5002端口
docker run -dp 127.0.0.1:5001:5002 testdb testdb

docker run -dti --network=host --name storage -e TRACKER_SERVER=47.106.87.191:22122 -v /var/fdfs/storage:/var/fdfs -v /etc/localtime:/etc/localtime delron/fastdfs storage
```

```shell
#运行镜像，不退出容器  (-d 后台执行,-i 交互,-t 终端)
docker run -dit 镜像
```

- 交互容器应用

```plain
docker run -it --rm testdb
docker run -it --rm testdb ls -l /test
```



### 镜像 image

- 运行容器，使用**隔离的文件系统**，这个用户文件系统需要容器镜像来提供
- 必须包含**容器运行所需的所有依赖，配置，脚本，二进制文件等**，其他配置如环境变量，运行默认命令，其他元数据
- dockerfile作为脚本可以用来创建镜像，不要加后缀
- 构建镜像

加上 --pull 时，Dockerfile 文件中 的 From 基础镜像 可能不会使用本地已下载好的镜像，而是会去远程 docker 仓库检查 Dockerfile 中的基础镜像 是不是 latest 的，如果不是，就会下载 最新的镜像作为基础镜像。所以我理解的这个参数就是针对 Dockerfile 里的 From 后基础镜像设置的。

```
docker build -t <镜像名> .
docker build --pull -t <镜像名> . 
```

- 重命名镜像

```shell
docker tag IMAGEID(镜像id) REPOSITORY:TAG(仓库：标签)
```

- 保存镜像

```shell
docker save -o 绝对路径\名称.tar  镜像id:tag
```

- 运行镜像

```shell
docker load -i 绝对路径\名称.tar
```

- 删除镜像

```plain
docker rmi 镜像名/镜像id
# 强制
docker rmi -f 镜像名/镜像id
```

- 搜索公共仓库镜像

```plain
docker search mysql
```

- 拉取镜像

```
docker pull mysql
```

- 推送镜像至私人仓库

```
docker push mysql
```

- 查看镜像版本

```plain
docker tags mysql
```

- 登录仓库（只有登录后可以和远程仓库交互）

```plain
docker login -u 用户名 -p 密码 (镜像仓库地址)
docker login -u admin https://10.29.3.30:30004(根据提示再输入密码)
```

```plain
docker login -u admin -p changeme https://10.29.3.30:30004
```

有时后k8s中某个节点拉取不到镜像（或者到达pull上限），可以手动上传镜像到私有仓库，k8s就可以自动拉取私有仓库的镜像，注意可能需要改动k8s中的应用的yaml（kubectl edit xxx ）

- docker desktop登录仓库

注意在配置中加上如下配置

```plain
"insecure-registries": ["10.29.3.30:30004"]
```

- 推送镜像到远程仓库

```plain
#上传本地镜像myapache:v1到镜像仓库中。
docker push myapache:v1
```

- 推送镜像到私有仓库

```plain
#对镜像做标记，将其归入某一仓库
docker tag 2305e26d3c02 10.23.12.108:30004/rftest/prom_test_metrics:latest
#推送镜像
docker push 10.23.12.108:30004/rftest/prom_test_metrics:latest
```

btw：这里使用私有仓库，镜像名一定要tag私有仓库/命名空间

```go
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prom-test-exporter
  namespace: rftest
  labels:
    k8s-app: prom-test-exporter
spec:
  selector:
    matchLabels:
      k8s-app: prom-test-exporter
  template:
    metadata:
      labels:
        k8s-app: prom-test-exporter
    spec:
      containers:
      - name: prom-test-exporter
      #这里私有仓库必须由tag仓库地址/命名空间
        image: 10.23.12.108:30004/rftest/prom_test_metrics
        # args: ["-redis.addr", "redis://192.168.122.7:6379", "-redis.password", "123456"]
        ports:
        - containerPort: 32323
          name: http
```



- 过滤镜像



```plain
docker images -f "key=value"
```



key支持：



```plain
dangling：显示标记为空的镜像，值只有true和false
label：这个是根据标签进行过滤，其中lable的值，是docker在编译的时候配置的或者在Dockerfile中配置的
before：这个是根据时间来进行过滤，其中before的value表示某个镜像构建时间之前的镜像列表
since：跟before正好相反，表示的是在某个镜像构建之后构建的镜像
reference：这个是添加正则进行匹配
```



```plain
只显示镜像id -q
docker images -qf "key=value"
查找
docker images -qf reference=helloapp:*
```



```plain
删除所有空镜像
docker rmi $(docker images -qf dangling=true)
```



格式化展示



```plain
docker images --format
```



```plain
Placeholder		Description
.ID	        	Image ID
.Repository		Image repository
.Tag			Image tag
.Digest		 	Image digest
.CreatedAt		 Time when the image was created
.Size			Image disk size
```



```plain
zhouzhenyong@shizi-2 ~> docker images --format "{{.ID}}\t{{.Repository}}"
942d4dd9eb3c	test/isc-panda
450e15c7459f	<none>
644627b7fb2e	<none>
b3f353ae77d2	simonalong/isc-panda
d30e0389349f	simonalong/cheers2019
be0dbf01a0f3	mysql
a86647f0b376	docker/desktop-kubernetes
2a71ac5a1359	docker/kube-compose-installer
e116d2efa2ab	golang
a3562aa0b991	openj
```



### 打包项目流程



-  在我们的项目目录下，新建dockerfile（无后缀）文件 
-  命令行打开文件的路径 

```shell
docker build -t getting-started:latest .
```


		有很多文件在下载是因为，我们需要镜像初始文件
		复制我们的app到镜像，用yarn安装镜像的依赖
		文件里的CMD指定我们开启容器后的默认命令
		-t标识符 将命名镜像为getting-started, tag为latest
		注意最后的点指dockerfile所在目录
		--name 命名容器 
		-d 后台运行
		-p 绑定端口 **暴露出的服务端口**是3000，**容器内的端口**是80（注意跟k8s nodeport是相反的）
![img](http://myimg.go2flare.xyz/img/image-20220311011049850.png)
![img](http://myimg.go2flare.xyz/img/image-20220311010827244.png)
		docker/getting-started 镜像运行
		单字符简写 -dp 

-  如果更改了内容需要更新镜像重复上面的2个命令即可 



# 构建镜像



## dockerfile



### 含有gin框架的项目打包，可以使用scratch的方式写镜像，可以节省很多空间



```dockerfile
# 打包依赖阶段使用golang作为基础镜像
FROM golang:1.14 as builder

# # CGO_ENABLED禁用cgo 然后指定OS等，启用go module
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64 \
	GOPROXY="https://goproxy.cn,direct"

WORKDIR /web

# 拷贝所有文件到web
COPY . .

#go build
RUN go build main.go
# 运行时使用scratch作为基础镜像
FROM scratch as prod

WORKDIR /web
# 为了防止代码中请求https链接报错，我们需要将证书纳入到scratch中
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/cert

COPY --from=builder /web .
# 指定运行时环境变量
ENV GIN_MODE=release

EXPOSE 9000

ENTRYPOINT ["./main"]
```



### 也可以使用Makefile先将项目编译成二进制文件，在将二进制文件拷贝到docker最小环境中运行



```dockerfile
FROM alpine
ADD userOrder-srv /userOrder-srv
ENTRYPOINT [ "/userOrder-srv" ]
```



# helloapp CI/CD demo



## dockerfile



```dockerfile
# 打包依赖阶段使用golang作为基础镜像
FROM golang:1.14 as builder

# # CGO_ENABLED禁用cgo 然后指定OS等，启用go module
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64 \
	GOPROXY="https://goproxy.cn,direct"

WORKDIR /app

COPY . .

#并go build
RUN go build main.go
# 运行时使用scratch作为基础镜像
FROM scratch as prod

WORKDIR /app
# 为了防止代码中请求https链接报错，我们需要将证书纳入到scratch中
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/cert

COPY --from=builder /app .
# 指定运行时环境变量
ENV GIN_MODE=release \
    PORT=80

EXPOSE 80

ENTRYPOINT ["./main"]
```



## deployment.yaml



从镜像仓库中导出



```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2022-01-04T10:45:46Z"
  labels:
    dsp.daocloud.io/application: fd24436e-36c7-49e2-b74b-800ed7b48930
  name: hellotest
  namespace: rftest
  # resourceVersion: "123502704"
  # selfLink: /api/v1/namespaces/hellotest/services/hellotest
  # uid: dafa4752-d8fe-43a1-8a9c-51d1bd4fccba
spec:
  clusterIP: 172.31.164.253
  externalTrafficPolicy: Cluster
  ports:
    - name: tt
      nodePort: 32288
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    dsp.daocloud.io/application: fd24436e-36c7-49e2-b74b-800ed7b48930
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2022-01-04T10:45:46Z"
  generation: 1
#  labels:
#    dsp.daocloud.io/application: fd24436e-36c7-49e2-b74b-800ed7b48930
  name: hellotest
  namespace: rftest
  # resourceVersion: "123502720"
  # selfLink: /apis/apps/v1/namespaces/hellotest/deployments/hellotest
  # uid: c9a2379d-f537-4229-8e50-d677aff14ee4
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      dsp.daocloud.io/application: fd24436e-36c7-49e2-b74b-800ed7b48930
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        dsp.daocloud.io/application: fd24436e-36c7-49e2-b74b-800ed7b48930
    spec:
      containers:
        - image: __image_name__
          imagePullPolicy: Always
          name: hellotest
          ports:
            - containerPort: 80
              protocol: TCP
          resources:
            limits:
              cpu: 10m
              memory: 10695475200m
            requests:
              cpu: 10m
              memory: 10695475200m
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsConfig: {}
      dnsPolicy: ClusterFirst
      imagePullSecrets:
        - name: hellotest
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  conditions:
    - lastTransitionTime: "2022-01-04T10:45:46Z"
      lastUpdateTime: "2022-01-04T10:45:46Z"
      message: Deployment does not have minimum availability.
      reason: MinimumReplicasUnavailable
      status: "False"
      type: Available
    - lastTransitionTime: "2022-01-04T10:45:46Z"
      lastUpdateTime: "2022-01-04T10:45:46Z"
      message: ReplicaSet "hellotest-996bb6db7" is progressing.
      reason: ReplicaSetUpdated
      status: "True"
      type: Progressing
  observedGeneration: 1
  replicas: 1
  unavailableReplicas: 1
  updatedReplicas: 1

---
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyIxMC43LjI1My4yMDEiOnsidXNlcm5hbWUiOiJzb3Utc2giLCJwYXNzd29yZCI6IlJ1eWlVZVZBZzc0MzgiLCJhdXRoIjoiYzI5MUxYTm9PbEoxZVdsVlpWWkJaemMwTXpnPSJ9fX0=
kind: Secret
metadata:
  creationTimestamp: "2022-01-04T10:45:46Z"
  labels:
    dsp.daocloud.io/application: fd24436e-36c7-49e2-b74b-800ed7b48930
  name: hellotest
  namespace: rftest
  # resourceVersion: "123502706"
  # selfLink: /api/v1/namespaces/hellotest/secrets/hellotest
  # uid: ab77b4de-356e-4e04-8ab2-42601db9b7c7
type: kubernetes.io/dockerconfigjson

---
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUdNRENDQkJpZ0F3SUJBZ0lKQU1CVkpiU3RXZ0JKTUEwR0NTcUdTSWIzRFFFQkN3VUFNRzB4Q3pBSkJnTlYKQkFZVEFrTk9NUkV3RHdZRFZRUUlFd2hUYUdGdVoyaGhhVEVSTUE4R0ExVUVCeE1JVTJoaGJtZG9ZV2t4RVRBUApCZ05WQkFvVENFUmhiME5zYjNWa01SRXdEd1lEVlFRTEV3aEVZVzlEYkc5MVpERVNNQkFHQTFVRUF4TUpiRzlqCllXeG9iM04wTUI0WERURTNNRFV4TURBMk16UTFPVm9YRFRJM01EVXdPREEyTXpRMU9Wb3diVEVMTUFrR0ExVUUKQmhNQ1EwNHhFVEFQQmdOVkJBZ1RDRk5vWVc1bmFHRnBNUkV3RHdZRFZRUUhFd2hUYUdGdVoyaGhhVEVSTUE4RwpBMVVFQ2hNSVJHRnZRMnh2ZFdReEVUQVBCZ05WQkFzVENFUmhiME5zYjNWa01SSXdFQVlEVlFRREV3bHNiMk5oCmJHaHZjM1F3Z2dJaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQ0R3QXdnZ0lLQW9JQ0FRQzFab3laTFdIc2ozVGMKN09PejRxZUlDa1J0NTFOM1lJVkNqc2cxTnA1ZllLQWRzQStNRktWN3VWYnc5djcrbmZpUEwySjFoeGx1MzF2UAo0Zlliamp5ZlAyd0w0RXlOeVBxYjlhNG5nRWZxZmhxOUo1QitCTWd2cTNSMFNCQ05YTTE2Mm1pSzBuUmlSU1pUCmlvSTY0S29wdjFvWXFsaW5mR2RzcnlPdHM1UUpYMFN1TEZKekhIK1hpa3dzQVJSOWNjRkpGajZTazJHODllZkUKZ1FaY2hvZThnd3BqSW9zK2w4S1BEVkxEdDFzeVM4MUVZaFpLZXpSWmZVWmdyb0VhZU5iRVdmNjg5NGpRQUdhNwpsbklpL0RCTWJXYjcwN1FybG9mYkt4ZWorSE9OUHdzai9uZFB5VjM3SFVNOG5iMFM3eTJ5V3A0dFJVdm5QQ2R4CkRWUjlyYzRMUEhGbjB5SUZtTmRrNTU5Q3BTS01meHBoTjNzbkZXdEsxZG1ieis3R2svUlpwRi96VkYvMnpEM2UKRkVTWmVZOU9ucUt0Wk1VZlR4dy9oZW83dC9OZHFNdU50TVJJY294SHREdzZIU0pibHhwTkxIcnh5M0l3Z3lUZApBZVMvR3Rjblg5T3hYRHFWUDJMN1RENlVuK05GRVBSbHhqTi9EVzRRS1gvdGg4T2FNTU9PekVZdlhTUUl2TVg4ClRTb2x5c0pBNGhrR0crM09YVUNENzFwS0N0TTVTU2lPTzNYc0xQdm1YYWt6NXpNd0p3cXBPUyt6M01LM0s4K08KRFBOcld1ZExNcm40VVduRkx2SzJhakx3Q2xTYk5Rdzk0K0I0eVdqVkR5a21hNGpkUm1QSkVvTVBRN3NNTGRISQpHbHdEOWkxMGdpaUpiR3haclp1a0pIUlMvMzFQS3dJREFRQUJvNEhTTUlIUE1CMEdBMVVkRGdRV0JCUlZsajNlCk1mWk1KNGVnSEdGbHRmcVAwSWxBNVRDQm53WURWUjBqQklHWE1JR1VnQlJWbGozZU1mWk1KNGVnSEdGbHRmcVAKMElsQTVhRnhwRzh3YlRFTE1Ba0dBMVVFQmhNQ1EwNHhFVEFQQmdOVkJBZ1RDRk5vWVc1bmFHRnBNUkV3RHdZRApWUVFIRXdoVGFHRnVaMmhoYVRFUk1BOEdBMVVFQ2hNSVJHRnZRMnh2ZFdReEVUQVBCZ05WQkFzVENFUmhiME5zCmIzVmtNUkl3RUFZRFZRUURFd2xzYjJOaGJHaHZjM1NDQ1FEQVZTVzByVm9BU1RBTUJnTlZIUk1FQlRBREFRSC8KTUEwR0NTcUdTSWIzRFFFQkN3VUFBNElDQVFBYjRnenI1dXhLZEdWTjBRck9meWNvS1BiTDltN0hsazlYcDFOYgpiVXQzQ0FnbHhDemc0NkVqOTNZK2dOZHF6SmNoU2o3M3RIYmRMYzY0Zlh1R3Riemp4RU55aUcwTlFVUXlVdEVBCjFKUmhJY2RSaG1uZVpzNGNNdm9OVTVmbU4yRllVZGFFT3JoUkRHd3pzQks1MDgzVXNDRVBaelhxV1FVRUpWNlQKVTVoMmJQbHUxT3ZwdlhpQ0hENG5kOVVSa21pZkdGSWZHWk16enRjay9MQnVEWE4wdUltSW1mSXluM0hkK2FNRQpZaTk1N1NjVFhuSXVkK0dtOVRkZjZSRW14Z0pkQVhwUmZVRm9UOVRBVURIcFhGcTlHcW4xSmlHUlJxRWFVbWZ6Cmp5ek5DMXowQmtMK2JkOG5LTGpseURhMVdaNHRuYU1yMGZ0TFp4dldYeEJ0NjBDcVM2Rk1SekhTUHpPRUNUSjQKb1g4WjlsQnhBYkx3bTBjSUx2K2JHdGxOREwzbGlxK3h1ck5OQjJsOHdFcndNUTdoUEVFeG1wQ0VJRGcxNVBCQgpKb3A0bEpxNTlwVms4dytNbzJzR3psMVVrSE5yOUdRbi9MZ3pCNDFrdTEzcll4dCthWFN0eTYzVUM1dUc5SEtlCldmY2U1RXE4YkcyZmZlME45c2xLdmc3K0FMNFdiNEtFVjk5U2VmY0pqL3JrcitiN2xrbERBZjl5cVJLNzdwMHkKZkZLT3dtSTVadlVSQW9BZDRBN1R4cXMvNjRNUjNWREhlMWZiMzVUU2g5RjZhSm0wVWNkQ2I3MGcwUG01bERoRwpOTTEyNjlKUHUxNVQxRHNHbWxUVGxSOXNHUFR4QnM0QlkvRDVFdDNRdFYvS2tIWTVDSW9RZnk3SXNCdWdyeU1rCjZ1UmZOQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  namespace: eGhvcGUtdGVzdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkltUnVibTFSYUVkbVdtTlBUR2hzZHpVd1dEZE9UVWhhZFdKdlJVZHpaMDV2Y1hOcmFqTnNTM1ZVYUhNaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUo0YUc5d1pTMTBaWE4wSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXpaV055WlhRdWJtRnRaU0k2SW1SbFptRjFiSFF0ZEc5clpXNHRhemhxWjNvaUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNXVZVzFsSWpvaVpHVm1ZWFZzZENJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZ5ZG1salpTMWhZMk52ZFc1MExuVnBaQ0k2SWpreE5XWTVNRE00TFdJelpEVXROREk1WlMwNU1EVm1MVEEzTURJeFpXRmpOVE5pWmlJc0luTjFZaUk2SW5ONWMzUmxiVHB6WlhKMmFXTmxZV05qYjNWdWREcDRhRzl3WlMxMFpYTjBPbVJsWm1GMWJIUWlmUS50cEpTU2ktckhFd0ZFNWlzVTBDb3lZdXVtRGk1TmJ4QnE2TS1SeWlEYWp1eTh2TEcxdXFkbldubV9iMU1UcFNvU0ZlZXIyVlc0ZkNTNDNPNG5sbXlKSXhfVWc2ZlhrbHlSazRabHFoaElmUXBRSkRyWUtHOTNnODU5WVRvR01WY2p4djRUamR4U09XRE1HQjh4bmNWclVaU1JIS1NCUUhKUW1zNmZXWXZvZ1pldjlLRFBCVDFyOEdSREthOGh1allXMExGX0VZZndtT2JhcFVQR0JySkl3LWs4SWczTE9wWDh5WW1xLW5UTjJFREE4akQwNjZKQkVqMXZHQW5JTDhlaGZkR3A0WEVnZnNxWDV4Sl8xUGMzQms5ODhZUGd6dnFLY2tnSkZBY0NyZ0VoMGhKdFFnUUp2WUJjUzdKbDdFVTVTTG9iXzJHZVVWTmNoelNrRlJZaUE=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"ca.crt":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUdNRENDQkJpZ0F3SUJBZ0lKQU1CVkpiU3RXZ0JKTUEwR0NTcUdTSWIzRFFFQkN3VUFNRzB4Q3pBSkJnTlYKQkFZVEFrTk9NUkV3RHdZRFZRUUlFd2hUYUdGdVoyaGhhVEVSTUE4R0ExVUVCeE1JVTJoaGJtZG9ZV2t4RVRBUApCZ05WQkFvVENFUmhiME5zYjNWa01SRXdEd1lEVlFRTEV3aEVZVzlEYkc5MVpERVNNQkFHQTFVRUF4TUpiRzlqCllXeG9iM04wTUI0WERURTNNRFV4TURBMk16UTFPVm9YRFRJM01EVXdPREEyTXpRMU9Wb3diVEVMTUFrR0ExVUUKQmhNQ1EwNHhFVEFQQmdOVkJBZ1RDRk5vWVc1bmFHRnBNUkV3RHdZRFZRUUhFd2hUYUdGdVoyaGhhVEVSTUE4RwpBMVVFQ2hNSVJHRnZRMnh2ZFdReEVUQVBCZ05WQkFzVENFUmhiME5zYjNWa01SSXdFQVlEVlFRREV3bHNiMk5oCmJHaHZjM1F3Z2dJaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQ0R3QXdnZ0lLQW9JQ0FRQzFab3laTFdIc2ozVGMKN09PejRxZUlDa1J0NTFOM1lJVkNqc2cxTnA1ZllLQWRzQStNRktWN3VWYnc5djcrbmZpUEwySjFoeGx1MzF2UAo0Zlliamp5ZlAyd0w0RXlOeVBxYjlhNG5nRWZxZmhxOUo1QitCTWd2cTNSMFNCQ05YTTE2Mm1pSzBuUmlSU1pUCmlvSTY0S29wdjFvWXFsaW5mR2RzcnlPdHM1UUpYMFN1TEZKekhIK1hpa3dzQVJSOWNjRkpGajZTazJHODllZkUKZ1FaY2hvZThnd3BqSW9zK2w4S1BEVkxEdDFzeVM4MUVZaFpLZXpSWmZVWmdyb0VhZU5iRVdmNjg5NGpRQUdhNwpsbklpL0RCTWJXYjcwN1FybG9mYkt4ZWorSE9OUHdzai9uZFB5VjM3SFVNOG5iMFM3eTJ5V3A0dFJVdm5QQ2R4CkRWUjlyYzRMUEhGbjB5SUZtTmRrNTU5Q3BTS01meHBoTjNzbkZXdEsxZG1ieis3R2svUlpwRi96VkYvMnpEM2UKRkVTWmVZOU9ucUt0Wk1VZlR4dy9oZW83dC9OZHFNdU50TVJJY294SHREdzZIU0pibHhwTkxIcnh5M0l3Z3lUZApBZVMvR3Rjblg5T3hYRHFWUDJMN1RENlVuK05GRVBSbHhqTi9EVzRRS1gvdGg4T2FNTU9PekVZdlhTUUl2TVg4ClRTb2x5c0pBNGhrR0crM09YVUNENzFwS0N0TTVTU2lPTzNYc0xQdm1YYWt6NXpNd0p3cXBPUyt6M01LM0s4K08KRFBOcld1ZExNcm40VVduRkx2SzJhakx3Q2xTYk5Rdzk0K0I0eVdqVkR5a21hNGpkUm1QSkVvTVBRN3NNTGRISQpHbHdEOWkxMGdpaUpiR3haclp1a0pIUlMvMzFQS3dJREFRQUJvNEhTTUlIUE1CMEdBMVVkRGdRV0JCUlZsajNlCk1mWk1KNGVnSEdGbHRmcVAwSWxBNVRDQm53WURWUjBqQklHWE1JR1VnQlJWbGozZU1mWk1KNGVnSEdGbHRmcVAKMElsQTVhRnhwRzh3YlRFTE1Ba0dBMVVFQmhNQ1EwNHhFVEFQQmdOVkJBZ1RDRk5vWVc1bmFHRnBNUkV3RHdZRApWUVFIRXdoVGFHRnVaMmhoYVRFUk1BOEdBMVVFQ2hNSVJHRnZRMnh2ZFdReEVUQVBCZ05WQkFzVENFUmhiME5zCmIzVmtNUkl3RUFZRFZRUURFd2xzYjJOaGJHaHZjM1NDQ1FEQVZTVzByVm9BU1RBTUJnTlZIUk1FQlRBREFRSC8KTUEwR0NTcUdTSWIzRFFFQkN3VUFBNElDQVFBYjRnenI1dXhLZEdWTjBRck9meWNvS1BiTDltN0hsazlYcDFOYgpiVXQzQ0FnbHhDemc0NkVqOTNZK2dOZHF6SmNoU2o3M3RIYmRMYzY0Zlh1R3Riemp4RU55aUcwTlFVUXlVdEVBCjFKUmhJY2RSaG1uZVpzNGNNdm9OVTVmbU4yRllVZGFFT3JoUkRHd3pzQks1MDgzVXNDRVBaelhxV1FVRUpWNlQKVTVoMmJQbHUxT3ZwdlhpQ0hENG5kOVVSa21pZkdGSWZHWk16enRjay9MQnVEWE4wdUltSW1mSXluM0hkK2FNRQpZaTk1N1NjVFhuSXVkK0dtOVRkZjZSRW14Z0pkQVhwUmZVRm9UOVRBVURIcFhGcTlHcW4xSmlHUlJxRWFVbWZ6Cmp5ek5DMXowQmtMK2JkOG5LTGpseURhMVdaNHRuYU1yMGZ0TFp4dldYeEJ0NjBDcVM2Rk1SekhTUHpPRUNUSjQKb1g4WjlsQnhBYkx3bTBjSUx2K2JHdGxOREwzbGlxK3h1ck5OQjJsOHdFcndNUTdoUEVFeG1wQ0VJRGcxNVBCQgpKb3A0bEpxNTlwVms4dytNbzJzR3psMVVrSE5yOUdRbi9MZ3pCNDFrdTEzcll4dCthWFN0eTYzVUM1dUc5SEtlCldmY2U1RXE4YkcyZmZlME45c2xLdmc3K0FMNFdiNEtFVjk5U2VmY0pqL3JrcitiN2xrbERBZjl5cVJLNzdwMHkKZkZLT3dtSTVadlVSQW9BZDRBN1R4cXMvNjRNUjNWREhlMWZiMzVUU2g5RjZhSm0wVWNkQ2I3MGcwUG01bERoRwpOTTEyNjlKUHUxNVQxRHNHbWxUVGxSOXNHUFR4QnM0QlkvRDVFdDNRdFYvS2tIWTVDSW9RZnk3SXNCdWdyeU1rCjZ1UmZOQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K","namespace":"eGhvcGUtdGVzdA==","token":"ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkltUnVibTFSYUVkbVdtTlBUR2hzZHpVd1dEZE9UVWhhZFdKdlJVZHpaMDV2Y1hOcmFqTnNTM1ZVYUhNaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUo0YUc5d1pTMTBaWE4wSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXpaV055WlhRdWJtRnRaU0k2SW1SbFptRjFiSFF0ZEc5clpXNHRhemhxWjNvaUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNXVZVzFsSWpvaVpHVm1ZWFZzZENJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZ5ZG1salpTMWhZMk52ZFc1MExuVnBaQ0k2SWpreE5XWTVNRE00TFdJelpEVXROREk1WlMwNU1EVm1MVEEzTURJeFpXRmpOVE5pWmlJc0luTjFZaUk2SW5ONWMzUmxiVHB6WlhKMmFXTmxZV05qYjNWdWREcDRhRzl3WlMxMFpYTjBPbVJsWm1GMWJIUWlmUS50cEpTU2ktckhFd0ZFNWlzVTBDb3lZdXVtRGk1TmJ4QnE2TS1SeWlEYWp1eTh2TEcxdXFkbldubV9iMU1UcFNvU0ZlZXIyVlc0ZkNTNDNPNG5sbXlKSXhfVWc2ZlhrbHlSazRabHFoaElmUXBRSkRyWUtHOTNnODU5WVRvR01WY2p4djRUamR4U09XRE1HQjh4bmNWclVaU1JIS1NCUUhKUW1zNmZXWXZvZ1pldjlLRFBCVDFyOEdSREthOGh1allXMExGX0VZZndtT2JhcFVQR0JySkl3LWs4SWczTE9wWDh5WW1xLW5UTjJFREE4akQwNjZKQkVqMXZHQW5JTDhlaGZkR3A0WEVnZnNxWDV4Sl8xUGMzQms5ODhZUGd6dnFLY2tnSkZBY0NyZ0VoMGhKdFFnUUp2WUJjUzdKbDdFVTVTTG9iXzJHZVVWTmNoelNrRlJZaUE="},"kind":"Secret","metadata":{"annotations":{"kubernetes.io/service-account.name":"default","kubernetes.io/service-account.uid":"915f9038-b3d5-429e-905f-07021eac53bf"},"creationTimestamp":"2021-04-12T09:13:58Z","name":"default-token-k8jgz","namespace":"hellotest"},"type":"kubernetes.io/service-account-token"}
    kubernetes.io/service-account.name: default
    kubernetes.io/service-account.uid: 915f9038-b3d5-429e-905f-07021eac53bf
  creationTimestamp: "2021-04-12T09:13:58Z"
  name: default-token-k8jgz
  namespace: rftest
  # resourceVersion: "123443183"
  # selfLink: /api/v1/namespaces/hellotest/secrets/default-token-k8jgz
  # uid: 5116d698-60b2-4ec1-a604-c29b848674ac
type: kubernetes.io/service-account-token
```



## gitlab CI



导入cicd_repo/templates/helloapp-pipeline.yml



```yaml
include:
  - project: 'runfeng.lin/gitlab-ci_service'
    ref: master
    file: 'cicd_repo/templates/helloapp-pipeline.yml'
```



cicd_repo/templates/helloapp-pipeline.yml



```yaml
#导入job脚本
include:
  - project: 'runfeng.lin/gitlab-ci_service'
    ref: master
    file: 'cicd_repo/jobs/buildimage.yml'
  - project: 'runfeng.lin/gitlab-ci_service'
    ref: master
    file: 'cicd_repo/jobs/pushimage.yml'
  - project: 'runfeng.lin/gitlab-ci_service'
    ref: master
    file: 'cicd_repo/jobs/deployimage.yml'

variables:
  CACHE_DIR: "target/"
  # 配置
  DOCKER_USER: "admin"
  DOCKER_PW: "daocloud"
  DEPLOY_REGISTRY: "10.7.253.201" #镜像仓库
  DEPLOY_SERVER: "root@10.7.253.11" #部署的服务器
  NAME_SPACE: "xhope-test"
  APP_NAME: "helloapp"

  # 构建镜像
  DOCKER_FILE_PATH: "./dockerfile"
  IMAGE_NAME: "${DEPLOY_REGISTRY}/${NAME_SPACE}/$APP_NAME:${CI_COMMIT_SHORT_SHA}"
  BUILD_IMAGE: "docker build --pull -t $IMAGE_NAME ."
  PUSH_IMAGE: "docker push $IMAGE_NAME"
  REMOVE_IMAGE: "docker rmi $IMAGE_NAME"

  #部署镜像
  DEPLOYMENT_YAML_PATH: "/root/deployment/user_deploy"

cache:
  paths:
    - ${CACHE_DIR}

# 设置状态
stages:
  - build_image
  - push_image
  - deploy_image

# 设置的状态所执行的工作
build_image:
  stage: build_image
  extends: .build_image

push_image:
  stage: push_image
  extends: .push_image

deploy_image:
  stage: deploy_image
  extends: .deploy_image
```



jobs



cicd_repo/jobs/buildimage.yml



```yaml
 .build_image:
  stage: build_image
  script:
    - docker info
    - docker login -u ${DOCKER_USER} -p ${DOCKER_PW} ${DEPLOY_REGISTRY} #登录仓库
    - docker build -t ${IMAGE_NAME} -f ${DOCKER_FILE_PATH} . #构建镜像
    - docker images
```



cicd_repo/jobs/pushimage.yml



```yaml
.push_image:
  stage: push_image
  script:
    # 上传镜像到仓库
    - docker push ${IMAGE_NAME}
    - docker rmi ${IMAGE_NAME}
    - docker images
```



cicd_repo/jobs/deployimage.yml



```go
.deploy_image:
  stage: deploy_image
  image: ${IMAGE_NAME}
  script:
    - sed -i "s#__image_name__#${IMAGE_NAME}#g" deployment.yaml #替换deployment中的变量
    - cat deployment.yaml #查看deployment.yaml文件
    - scp deployment.yaml ${DEPLOY_SERVER}:${DEPLOYMENT_YAML_PATH} #将文件传输至部署应用的服务器
    - ssh $DEPLOY_SERVER "kubectl apply -f ${DEPLOYMENT_YAML_PATH}/deployment.yaml" #在部署应用的服务器远程执行k8s部署yaml
  after_script:
    - ssh $DEPLOY_SERVER "kubectl get pod -n xhope-test"
```



# docker 命令



### 容器生命周期管理



- [run](https://www.runoob.com/docker/docker-run-command.html)
- [start/stop/restart](https://www.runoob.com/docker/docker-start-stop-restart-command.html)
- [kill](https://www.runoob.com/docker/docker-kill-command.html)
- [rm](https://www.runoob.com/docker/docker-rm-command.html)
- [pause/unpause](https://www.runoob.com/docker/docker-pause-unpause-command.html)
- [create](https://www.runoob.com/docker/docker-create-command.html)
- [exec](https://www.runoob.com/docker/docker-exec-command.html)



### 容器操作



- [ps](https://www.runoob.com/docker/docker-ps-command.html)
- [inspect](https://www.runoob.com/docker/docker-inspect-command.html)
- [top](https://www.runoob.com/docker/docker-top-command.html)
- [attach](https://www.runoob.com/docker/docker-attach-command.html)
- [events](https://www.runoob.com/docker/docker-events-command.html)
- [logs](https://www.runoob.com/docker/docker-logs-command.html)
- [wait](https://www.runoob.com/docker/docker-wait-command.html)
- [export](https://www.runoob.com/docker/docker-export-command.html)
- [port](https://www.runoob.com/docker/docker-port-command.html)



### 容器rootfs命令



- [commit](https://www.runoob.com/docker/docker-commit-command.html)
- [cp](https://www.runoob.com/docker/docker-cp-command.html)
- [diff](https://www.runoob.com/docker/docker-diff-command.html)



### 镜像仓库



-  [login](https://www.runoob.com/docker/docker-login-command.html) 
-  [pull](https://www.runoob.com/docker/docker-pull-command.html) 
-  [push](https://www.runoob.com/docker/docker-push-command.html) 

```plain
docker push : 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库

语法
docker push [OPTIONS] NAME[:TAG]
OPTIONS说明：

--disable-content-trust :忽略镜像的校验,默认开启

实例
上传本地镜像myapache:v1到镜像仓库中。

docker push myapache:v1
```

 

-  [search](https://www.runoob.com/docker/docker-search-command.html) 



### 本地镜像管理



- [images](https://www.runoob.com/docker/docker-images-command.html)
- [rmi](https://www.runoob.com/docker/docker-rmi-command.html)
- [tag](https://www.runoob.com/docker/docker-tag-command.html)
- [build](https://www.runoob.com/docker/docker-build-command.html)
- [history](https://www.runoob.com/docker/docker-history-command.html)
- [save](https://www.runoob.com/docker/docker-save-command.html)
- [load](https://www.runoob.com/docker/docker-load-command.html)
- [import](https://www.runoob.com/docker/docker-import-command.html)



### info|version



- [info](https://www.runoob.com/docker/docker-info-command.html)
- version



# docker proxy

Docker：网络模式详解

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/21986264/1651558006410-eb086f53-33db-4097-87d4-96650a43ae42.jpeg)

安装Docker时，它会自动创建三个网络，bridge（创建容器默认连接到此网络）、 none 、host

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/21986264/1651558011740-526c8533-b378-4f22-a6ff-2f2ac794db0c.jpeg)

docker run创建Docker容器时，可以用 --net 选项指定容器的网络模式 ：

```plain
host模式：使用 --net=host 指定。
none模式：使用 --net=none 指定。
bridge模式：使用 --net=bridge 指定，默认设置。
container模式：使用 --net=container:NAME_or_ID 指定
```

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/21986264/1651558036896-35a62d83-6788-4284-84ba-427c4efee606.jpeg)

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/21986264/1651558044035-0e69b794-4663-48f9-938a-6e46473511e0.jpeg)

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/21986264/1651558050262-4f663833-fb4f-47bd-8ee6-d9f339000ec4.jpeg)

```plain
启动docker engine后，会在主机上创建一个名为docker0的虚拟网桥，此主机上启动的Docker容器会连接到
这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个
二层网络中。
从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。在主机上创建一对虚拟网卡
veth pair设备，Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡）
，另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。
 
[root@g15-6f-81-238 ~]# ifconfig
'''
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
         inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
'''
 
[root@g15-6f-81-238 ~]# docker inspect d1872d45b01d | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.6",
            
为了实现上述功能，docker主要用到了linux的Bridge、Network Namespace、VETH (虚拟网卡的接口对 Virtual Enternet device)。
docker0网关就是通过Bridge实现的。
Network Namespace是网络命名空间，通过Network Namespace可以建立一些完全隔离的网络栈。
比如通过docker network create xxx就是在建立一个Network Namespace。
VETH是虚拟网卡的接口对，可以把两端分别接在两个不同的Network Namespace中，实现两个原本隔离的Network Namespace的通信。
所以总结起来就是：
    Network Namespace做了容器和宿主机的网络隔离，
    Bridge分别在容器和宿主机建立一个网关，
    然后再用VETH将容器和宿主机两个网络空间连接起来。
```

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/21986264/1651558101696-bf8056df-0052-4cfa-98ed-9280b514b96e.jpeg)





[
](https://blog.csdn.net/weixin_34608222/article/details/113537311)



## docker的proxy容器的ip就是在docker0虚拟网卡上递增的



```plain
172.17.0.1（docker 虚拟网卡）
172.17.0.2（容器consul）
172.17.0.3（helloapp)
172.17.0.4（yzf-web)
```



## docker proxy详解



知识回顾：



server绑定网卡不一定只可以绑定一个，比如只绑localhost:8000，172.17.0.1:8000



**可以绑定0.0.0.0来接收所有网卡请求**



```plain
0.0.0.0:8000（ipv4）
[::]:8000（ipv6）
```



```plain
-p, --publish list      Publish a container's port(s) to the host推送容器的端口映射宿主机端口
```



- **宿主机请求微服务网络请求链路过程**：



request 47.106.87.191:8500(外网) -> 172.29.113.151:8500(内网) -> 172.17.0.1(docker-proxy虚拟网卡) -> 172.17.0.7(容器网卡)



![img](http://myimg.go2flare.xyz/img/image-20220317230920597.png)



![img](http://myimg.go2flare.xyz/img/image-20220317224933470.png)



![img](http://myimg.go2flare.xyz/img/image-20220317232532415.png)



![img](http://myimg.go2flare.xyz/img/image-20220317233234870.png)



- **微服务请求微服务请求链路过程**：



1.captcha微服务注册consul



出现以下问题：



![img](http://myimg.go2flare.xyz/img/image-20220317235224176.png)



```go
func main() {
	//初始化consul注册服务
	regOpt := registry.Option(func(options *registry.Options){
		options.Addrs= []string{"localhost:8500"}
        //改为 options.Addrs= []string{"172.17.0.1:8500"}
	})
	consulRegistry := consul.NewRegistry(regOpt)

	//初始化reds
	model.InitRedis()

	// New Service
	service := micro.NewService(
		//指定微服务连接服务发现的端口
		micro.Address(":52666"), //防止随机生成port，注意这里要用0.0.0.0
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



![img](http://myimg.go2flare.xyz/img/image-20220318000031858.png)



2.WEB微服务请求captcha微服务问题：



出现以下报错



```shell
[GIN] 2022/03/17 - 15:35:42 | 304 |      74.437µs |  203.168.29.113 | GET      "/home/favicon.ico"
2022/03/17 15:35:47 [ERR] consul.watch: Watch (type: services) errored: Get "http://localhost:8500/v1/catalog/services": dial tcp 127.0.0.1:8500: connect: connection refused, retry in 20s
```



排查到请求微服务



```go
//初始化micro
func InitMicro() client.Client{
//	初始化客户端
	regOpt := registry.Option(func(options *registry.Options){
		//请求微服务的配置是localhost网卡，请求会发不出去
        options.Addrs= []string{"localhost:8500"}
        //改为 options.Addrs = []string{"172.17.0.1:8500"}
	})
	consulRegistry := consul.NewRegistry(regOpt)
	// consulRegistry := consul.NewRegistry()
	microClient := micro.NewService(
		micro.Registry(consulRegistry),
	)
	return microClient.Client()
}
```



![img](http://myimg.go2flare.xyz/img/image-20220317234545702.png)



- **微服务正确链路调用**



![img](http://myimg.go2flare.xyz/img/image-20220318001754972.png)



# docker安装应用



## nginx

运行正常的容器

```
docker run --name nginx -dp 82:80 nginx:latest
```

/etc/nginx中有配置文件，需要从正常的容器中拷贝，

```
docker cp nginx:/etc/nginx /etc/docker/nginx/config/
docker cp nginx:/usr/share/nginx/html/ /etc/docker/nginx/data/
docker cp nginx:/var/log/nginx /etc/docker/nginx/log/
```

容器配置文件挂载

```
docker run --name nginx -p 80:80 \
-v /etc/docker/nginx/config/nginx/:/etc/nginx \
-v /etc/docker/nginx/data/shop:/usr/share/nginx/shop \
-v /etc/docker/nginx/data/manager:/usr/share/nginx/manager \
-v /etc/docker/nginx/logs/:/var/log/nginx \
-d nginx:latest
```

用自己的商城页面/docker/nginx/data/html替换掉nginx welcome即可



## mysql 8.0.11



获取默认生成的配置文件
先通过如下命令，运行一个容器，名字叫mysql:



```plain
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql --default-authentication-plugin=mysql_native_password
```



容器运行之后，通过如下命令把my.cnf文件拷贝到上面创建的conf目录下



```plain
docker cp mysql:/etc/mysql/my.cnf E:/docker/mysql/conf/
```



配置文件，可直接覆盖



```plain
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
 
#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html
 
[mysql]
 
#设置mysql客户端默认字符集
default-character-set=utf8
 
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
 
#服务端使用的字符集默认为8比特编码的latin1字符集
character_set_server = utf8
 
#创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
 
#设置不区分大小写
# 必须在安装好MySQL后 修改mySQL配置文件设置为不敏感，一旦启动后，再设置是无效的，而且启动报错；
# 如果已经晚了，那必须把MySQL数据库文件全部 删除，修改配置文件再启动。
lower_case_table_names=1

#是否开启慢查询日志
slow_query_log =ON
#慢查询的阈值，单位秒
long_query_time=3
#是否记录未使用索引的查询语句，记录在慢查询日志
log-queries-not-using-indexes=OFF
#错误日志
log-error=/var/lib/mysql/error.log
#慢查询日志
slow_query_log_file=/var/lib/mysql/slowquery.log
```



在主机创建以下文件



```plain
├─conf
├─data
└─mysql-files
```



c:/doc_repo可自定义



```plain
docker run --name mysql -p 3306:3306 -v /c/doc_repo/mysql/data:/var/lib/mysql/ -v /c/doc_repo/mysql/conf/my.cnf:/etc/mysql/my.cnf -v /c/doc_repo/mysql/mysql-files:/var/lib/mysql-files/ -e MYSQL_ROOT_PASSWORD=4.234.23123aa -d mysql:8.0.11 --default-authentication-plugin=mysql_native_password
```



测试



```plain
docker exec -it mysql bash
```



```plain
mysql -uroot -p4.234.23123aa
```



这一步看博客不知道干嘛先不管



```plain
# 登录mysql
mysql -uroot -p123456
# 切换到mysql数据库
use mysql;
# 查询user表，如果有两个root用户，删除掉host=%的root数据，再修改localhost为%
select host, user, authentication_string, plugin from user;
delete from user where host="%" and user="root";
update user set host = '%' where user = 'root';
alter user 'root'@'%' identified with mysql_native_password by '123456';
# 立即生效
FLUSH PRIVILEGES;
```



REF



```plain
https://blog.csdn.net/baodong3930/article/details/102248390
https://blog.csdn.net/zt102545/article/details/108226332
```



## consul

拉取Consul镜像



```shell
docker pull consul # 默认拉取latest
docker pull consul:1.6.1 # 拉取指定版本
```



安装并运行

```shell
docker run -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600 --restart=always --name=consul consul:latest agent -server -bootstrap -ui -node=node1 -client='0.0.0.0'
# windows
docker run -d --name=consul -p 8500:8500 consul agent -server -bootstrap -ui -client 0.0.0.0
```



bobby方式

```json
docker run -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600/udp consul consul agent -dev -c1ient=0.0.0.0
```

集群部署

https://www.cnblogs.com/kazihuo/p/10710463.html

**server**

```json
docker run -d --name=consul --net=host -e CONSUL_BIND_INTERFACE=eth0 consul agent -server=true -client=0.0.0.0 -bind=192.168.10.11 -ui -bootstrap-expect=2
docker run -d --name=consul --net=host -e CONSUL_BIND_INTERFACE=eth0 consul agent -server=true -client=0.0.0.0 -bind=172.29.113.152 -ui -bootstrap-expect=1 -node=120.24.221.188
```

需要绑定docker外部的网卡

### 参数说明

1、eth0代表服务器卡网，也可能是ens33，ens160等，根据服务器信息传入相应参数即可；

2、通过bind本机ip内网网卡，改变了consul服务对外的ip地址，默认是172.17.0.2；查看命令是 docker inspect --format '{{ .NetworkSettings.IPAddress }}' consul1 ，consul1是对应的容器名称；

3、-node=120.24.221.188是consul中对节点的命名

4、**--net=host --将容器需要映射的端口全部映射到物理机上，并且端口保持不变**；

5、当需要指定配置文件和数据目录时，可用如下命令启动：

```json
docker run -d --name=consul1 -v /consulconfig:/config/file -config-dir=/config/file -date-dir=/tmp/consul --net=host -e CONSUL_BIND_INTERFACE=eth0 consul agent -server=true -client=0.0.0.0 -bind=192.168.10.11 -ui -bootstrap-expect=2
```

查看成员及IP

```json
 docker exec cb63d6139c16 consul members
```

client

```json
docker run -d --name=consul --net=host -e CONSUL_BIND_INTERFACE=eth0 consul agent -server -client=0.0.0.0 -ui -bootstrap-expect=1 -retry-join=120.24.221.188 -node=47.106.87.191
docker run -d --name=consul --net=host -e CONSUL_BIND_INTERFACE=eth0 consul agent -server -client=0.0.0.0 -ui -retry-join=172.29.113.152 -node=47.106.87.191
```

- agent: 表示启动 Agent 进程。
- server：表示启动 Consul Server 模式
- client：表示启动 Consul Cilent 模式。
- bootstrap：表示这个节点是 Server-Leader ，每个数据中心只能运行一台服务器。技术角度上讲 Leader 是通过 Raft 算法选举的，但是集群第一次启动时需要一个引导 Leader，在引导群集后，建议不要使用此标志。
- ui：表示启动 Web UI 管理器，默认开放端口 8500，所以上面使用 Docker 命令把 8500 端口对外开放。
- node：节点的名称，集群中必须是唯一的，默认是该节点的主机名。
- client：consul服务侦听地址，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1所以不对外提供服务，如果你要对外提供服务改成0.0.0.0
- join：表示加入到某一个集群中去。 如：-json=192.168.0.11。



测试

```plain
http://47.106.87.191:8500
```

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650912742132-f5a1e2b8-3c02-44d7-94e0-02041af1d30f.png)

占用内存大概

```json
80m
```

## nacos

```plain
docker run --name nacos-standalone -e MODE=standalone -e JVM_XMS=512m -e JVM_XMX=512M -e JVM_XMN=256m -p 8848:8848 -d nacos/nacos-server:latest
# 小内存版
docker run --name nacos-standalone -e MODE=standalone -e JVM_XMS=128m -e JVM_XMX=128M -e JVM_XMN=128m -p 8848:8848 -d nacos/nacos-server:latest
```



实测后台占用CPU100m内，但占用内存500m左右



```plain
#进入容器
docker exec -it nacos-standalone /bin/sh
```



进入主页



```plain
http://47.106.87.191:8848/nacos/index.html
```

默认账户密码

```json
nacos/nacos
```

占用内存大概

```json
700m
```

### 1.命名空间

可以隔离配置集，将某些配置集放在某个命名空间下，命名空间我们一般来区分微服务

### 2.组

可以隔离不同的命名空间，相当于不同的环境，可以建立开发环境，测试环境，生产环境

### 3.dataid-配置集

一个配置集就是一个配置文件，可以很灵活



## Kong

两种数据库postgres，cassandra，选择一种安装即可，以下命令为安装所需数据库和依赖，应为如果kong安装在docker中，可能因为docker proxy导致会出现很多不必要的复杂的错误需要排除，所以我们将其安装在主机中

```json
docker run -d --name kong-database \
-p 5432:5432 \
-e "POSTGRES_USER=kong" \
-e "POSTGRES_DB=kong" \
-e "POSTGRES_PASSWORD=kong" \
-e "POSTGRES_DB=kong" postgres:12

docker run --rm\
-e "KONG_DATABASE=postgres"\
-e "KONG_PG_H0ST=120.24.221.188"\
-e "KONG_PG_PASSWORD=kong"\
-e "POSTGRES_USER=kong"\
-e "KONG_CASSANDRA_CONTACT_POINTS=kong-database"\
kong kong migrations bootstrap
```

## 下载和安装

可以到这里找下载链接：https:/docs.konghq.com/install/centos/

```json
 curl $(rpm --eval "https://download.konghq.com/gateway-2.x-centos-%{centos_ver}/config.repo") | sudo tee /etc/yum.repos.d/kong.repo
 sudo yum install kong-enterprise-edition-2.8.1.0
```

### 编辑kong配置

关闭防火墙、重启docker 很重要

```json
systemctl stop firewalld.service
systemctl restart docker
```

 

修改如下内容

```json
cp /etc/kong/kong.conf.default /etc/kong/kong.conf
vim /etc/kong/kong.conf

#诊改如下内容
database = postgres
pg_host = 120.24.221.188  #这里得配置对外ip地址不能是127.0.0.1
pg_port = 5432 #Port of the Postgres server.
pg_timeout = 5000 # Defines the timeout (in ms),for connecting,reading and writing.
pg_user = kong # Postgres user.
pg_password kong # Postgres user's password.
pg_database kong # The database name to connect to.
dns_resolver = 172.29.113.152:8600 #这个配置很重要，配置的是consul的dns端口，配置默认是8600
admin_listen = 0.0.0.0:8001 
proxy_listen = 0.0.0.0:8000 reuseport backlog=16384 0.0.0.0:8444
```

运行

```json
kong migrations bootstrap up -c /etc/kong/kong.conf
kong start -c /etc/kong/kong.conf
```

需要开放一些端口

```json
#添加防火墙规则
firewall-cmd --zone=public --add-port=8001/tcp --permanent
firewall-cmd --zone=public --add-port=8000/tcp --permanent
sudo firewall-cmd --reload
```



重新修改配置运行，kill掉进行，重新执行运行那一步

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650937086405-2f0c7ba7-f8d6-487e-a8e3-45b2d7432b9d.png)

安装konga管理，好比navicat，负责kong的图形化界面管理

```json
安装konga
docker run -d -p 1337:1337 --name konga pantsel/konga
```

测试

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650711091333-5bccdbd0-6268-4e94-8455-fcfdbf5a64fd.png)

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650711096337-200cc67f-17ad-4ee6-b473-21255acd9c2b.png)

登录连接kong admin时有点小问题，阿里云安全组等把自己的请求也给打开。。。

8001:kong的管理的端口

8000:用户访问

1337:konga地址

端口关系

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1651593328475-106c7dbc-0385-44ee-b800-95c0025a4aa7.png)

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650721387037-0a43b918-da14-41ef-b3e2-aec22229a78b.png)

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650722009930-57d270b0-a5da-473b-8bb7-7a2f0ac28469.png)

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650722110598-032ba370-8bdf-4f0e-8548-62f4e21fafaa.png)

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650722826868-3938041c-427d-4374-8ba9-68ebdaf34014.png)

前面将consul的配置集成到Kong里面了，所以这里用consul给每个微服务分配的域名访问也可以访问到微服务，因为kong可以实现请求的负载均衡，consul只是健康检查，而且启动一个微服务的多个实例的时候，通过kong请求的consul中的微服务也可以实现负载均衡。

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650723033076-1e6a0de6-806e-4880-bee7-7f30fdb8bf0d.png)

### 3.配置jwt

通用认证

一般情况下，上游PI服务都需要客户端有身份认证，且不允许错误的认证或无认证的请求通过。认证插件可以实

现这一需求。这些插件的通用方案/流程如下：

1.向一个API或全局添加AUTH插件（此插件不作用于consumers):

2.创建一个consumer>对象：

3.为consumer提供指定的验证插件方案的身份验证凭据

4.现在，只要有请求进入Kong,都将检查其提供的身份验证凭据（取决于auth类型），如果该请求无法被验证或者验证失败，则请求会被锁定，不执行向上游服务转发的操作。

但是，上述的一般流程并不是总是有效的。譬如，当使用了外部验证方案（比如LDAP)时，KONG就不会（不需

要)对consumeri进行身份验证

Consumers

最简单的理解和配置consumer的方式是，将其于用户进行一映射，即一个consumer代表一个用户（或应用）。但是对于KONG而言，这些都无所谓。consume的核心原则是你可以为其添加插件，从而自定义他的请求行为。

所以，或许你会有一个手机APP应用，并为他的每个版本都定义石g9 nsumer,,又或者你又一个应用或几个应用，并为这些应用定义统一个consumer,这些都无所谓。这是一个模糊的概念，他叫做consumer,而不是user!万万要区分开来，且不可混淆。

匿名验证

首先需要创建一个Service来做上有服务，来匹配到相应的相应的转发的目的地，一个Service可以由多个Route,匹配到的Route都会转发给Service。

Service可以是一个实际的地址，也可以是kong内部提供的upstream object

**注意：新建jwt的时候key必须和实际生成的token中的payload中的iss的值保持一致**


![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650724696360-ca60600d-9cfd-4169-84c6-90124d0039a1.png)

## RocketMQ

docker-compose

```json
version: '3.5'
services:
  rmqnamesrv:
    image: foxiswho/rocketmq:server
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - ./logs:/opt/logs
      - ./store:/opt/store
    networks:
        rmq:
          aliases:
            - rmqnamesrv

  rmqbroker:
    image: foxiswho/rocketmq:broker
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - ./logs:/opt/logs
      - ./store:/opt/store
      - ./conf/broker.conf:/etc/rocketmq/broker.conf
    environment:
        NAMESRV_ADDR: "rmqnamesrv:9876"
        JAVA_OPTS: " -Duser.home=/opt"
        #JAVA_OPT_EXT: "-server -Xms256m -Xmx256m -Xmn256m"
        JAVA_OPT_EXT: "-server -Xms128m -Xmx128m -Xmn128m"
    command: mqbroker -c /etc/rocketmq/broker.conf
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqbroker

  rmqconsole:
    image: styletang/rocketmq-console-ng
    container_name: rmqconsole
    ports:
      - 8080:8080
    environment:
        JAVA_OPTS: "-Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false"
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqconsole

networks:
  rmq:
    name: rmq
    driver: bridge
```


conf/broker.json

```json
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.


# 所属集群名字
brokerClusterName=DefaultCluster

# broker 名字，注意此处不同的配置文件填写的不一样，如果在 broker-a.properties 使用: broker-a,
# 在 broker-b.properties 使用: broker-b
brokerName=broker-a

# 0 表示 Master，> 0 表示 Slave
brokerId=0

# nameServer地址，分号分割
# namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876

# 启动IP,如果 docker 报 com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <192.168.0.120:10909> failed
# 解决方式1 加上一句 producer.setVipChannelEnabled(false);，解决方式2 brokerIP1 设置宿主机IP，不要使用docker 内部IP
brokerIP1=120.78.192.25

# 在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

# 是否允许 Broker 自动创建 Topic，建议线下开启，线上关闭 ！！！这里仔细看是 false，false，false
autoCreateTopicEnable=true

# 是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

# Broker 对外服务的监听端口
listenPort=10911

# 删除文件时间点，默认凌晨4点
deleteWhen=04

# 文件保留时间，默认48小时
fileReservedTime=120

# commitLog 每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824

# ConsumeQueue 每个文件默认存 30W 条，根据业务情况调整
mapedFileSizeConsumeQueue=300000

# destroyMapedFileIntervalForcibly=120000
# redeleteHangedFileInterval=120000
# 检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
# 存储路径
# storePathRootDir=/home/ztztdata/rocketmq-all-4.1.0-incubating/store
# commitLog 存储路径
# storePathCommitLog=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/commitlog
# 消费队列存储
# storePathConsumeQueue=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/consumequeue
# 消息索引存储路径
# storePathIndex=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/index
# checkpoint 文件存储路径
# storeCheckpoint=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/checkpoint
# abort 文件存储路径
# abortFile=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/abort
# 限制的消息大小
maxMessageSize=65536

# flushCommitLogLeastPages=4
# flushConsumeQueueLeastPages=2
# flushCommitLogThoroughInterval=10000
# flushConsumeQueueThoroughInterval=60000

# Broker 的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写Master
# - SLAVE
brokerRole=ASYNC_MASTER

# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

# 发消息线程池数量
# sendMessageThreadPoolNums=128
# 拉消息线程池数量
# pullMessageThreadPoolNums=128
```

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650808275108-aec7d654-8ac9-4a2b-869c-fdc91a360f89.png)

## 通过docker安装elasticsearch

```plain
#新建es的config配置文件夹
mkdir -p /data/elasticsearch/config

#新建es的data目录
mkdir -p /data/elasticsearch/data

#给目录设置权限
chmod 777 -R /data/elasticsearch

#写入配置到elasticsearch.yml中
echo "http.host: 0.0.0.0">>/data/elasticsearch/config/elasticsearch.yml

#安装es
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS=" -Xms128m -Xmx256m" \
-v /data/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.10.1

#如果安装失败需要重启docker
systemctl restart docker
```



### 出现的问题



```plain
Exception in thread "main" SettingsException[Failed to load settings from [elasticsearch.yml]]; nested: ParsingException[Failed to parse object: expecting token of type [START_OBJECT] but found [VALUE_STRING]];
```



![img](http://myimg.go2flare.xyz/img/image-20220425002418266.png)



- 配置文件中冒号后面加空格
- [120.78.192.25:9200](http://120.78.192.25:9200/)



![img](http://myimg.go2flare.xyz/img/image-20220425002734892.png)



### 安装kibana



```plain
docker run -d --name kibana -e ELASTICSEARCH_HOSTS="http://120.78.192.25:9200" -p 5601:5601 kibana:7.10.1
```

# ![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650819002531-f5c36467-ab1b-4b24-b064-21268ebb7f59.png)

## 安装jaeger

```plain
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14250:14250 \
  -p 14268:14268 \
  -p 14269:14269 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.33
```

| Port  | Protocol | Component | Function                                                     |
| ----- | -------- | --------- | ------------------------------------------------------------ |
| 5775  | UDP      | agent     | 接受而非紧凑的节俭协议(不推荐使用，仅供传统客户使用) zipkin.thrift |
| 6831  | UDP      | agent     | 接受超越紧凑的节俭协议，节俭                                 |
| 6832  | UDP      | agent     | 在二进制节俭protocoljaeger.thrift接受                        |
| 5778  | HTTP     | agent     | 服务配置                                                     |
| 16686 | HTTP     | query     | 服务前端                                                     |
| 14268 | HTTP     | collector | 直接接受客户/jaeger.thrift                                   |
| 14250 | HTTP     | collector | 接受model.proto                                              |
| 9411  | HTTP     | collector | Zipkin兼容的端点(可选)                                       |

## 安装gitlab-ce

```
# 192.168.2.53 是我虚拟机的地址
docker run --name gitlab --hostname 192.168.2.45 -d -p 24:22 -p 80:80 -p 433:433 gitlab/gitlab-ce

# 映射
docker run --name gitlab --hostname 10.212.9.186 -v /c/doc_repo/gitlab-ce/gitlab.rb:/etc/gitlab/gitlab.rb -d -p 24:22 -p 80:80 -p 433:433 gitlab/gitlab-ce 
```

## 安装keycloak

```
docker run -dp 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=changeme quay.io/keycloak/keycloak:18.0.0 start-dev

docker run -dp 8088:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=changeme jboss/keycloak:11. start-dev
```

# 常见问题

## [docker内的应用访问宿主机上的mysql和Redis](https://www.cnblogs.com/java-bhp/p/11849045.html)



**背景：**宿主机部署MySQL、Redis，docker内部署tomcat、jdk



**需求：**tomcat内的应用访问宿主机的MySQL和Redis



**方法：**



1.  连接地址切记不能用localhost和127.0.0.1
   这些地址代表的都是容器内的系统，根本没有访问到宿主机，会一直报连接mysql/redis异常。 
2.  用docker的虚拟网卡地址
      	在宿主机查询网卡情况------ifconfig
      	docker0这块虚拟网卡的 inet 地址就是正确的本地ip(如172.17.0.1) 



![img](https://img2018.cnblogs.com/i-beta/1548895/201911/1548895-20191113144911984-1043109677.png)



## 连接宿主机mysql或redis



### mysql



宿主机的ip如何通过容器获取



```plain
service user connect mysql err Error 1130: Host '172.17.0.4' is not allowed
```



mysql和redis中配置有连接限制



```plain
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



```plain
2022/03/12 09:21:13 mysql配置连接成功，username=root, password=4.234.23123aa, host=172.17.0.1, port=3306, database=test, charset=utf8
```



### 容器中使用宿主机的redis



- 可以直接绑定所有网卡，这样来自所有网卡都可连接redis



```plain
bind 0.0.0.0
```



![img](http://myimg.go2flare.xyz/img/image-20220314011903320.png)



- 宿主机的conf绑定虚拟网卡的ip



网卡ip



```plain
127.0.0.1 172.17.0.1 172.18.0.1 172.29.113.151
```



conf中绑定这些网卡，则来自这些ip的连接请求都可通过



```plain
bind 172.17.0.1 127.0.0.1 172.18.0.1
```



测试结果



```plain
2022/03/12 10:21:55 redis配置连接成功，network=tcp, address=172.18.0.1:6379, password=4.234.23123

但是redis显示只有绑定一张网卡，所以要不就指定一张，要不就直接全部网卡
```



![img](http://myimg.go2flare.xyz/img/image-20220314012458526.png)



## 报sock permission denied

在用户权限下docker 命令需要 sudo 否则出现以下问题

![img](https://img-blog.csdnimg.cn/20200227175406444.png)

1. 单次授予，重启后失效

  ```
  sudo chmod 666 /var/run/docker.sock
  ```


2. 永久授予
    2.1. 将当前用户加入docker组

  ```
  sudo gpasswd -a $USER docker
  或
  sudo usermod -a -G docker $USER
  ```

​		2.2. 授予普通用户权限

```
# 查看docker.socket配置
cat /usr/lib/systemd/system/docker.socket
```


可以看到以下内容

```
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
```


其中ListenStream为unix domain socket文件路径，SocketMode为权限模式，SocketUser为所属用户，SocketGroup为所属组。
我们只需修改**SocketMode为0600**即可。
修改并保存完成后，重新加载守护并重启socket即可：

```
sudo systemctl daemon-reload
sudo systemctl restart docker.socket
```


再次使用docker发现不再报错，重启机器后仍然可用。



## scratch镜像问题



使用scratch镜像时，因为时空镜像，没有bash，所以mv，cp，rm等命令都不可用，只是提供一个环境将构建好的二进制文件运行起来的环境，如下是使用scratch镜像的大小



![img](http://myimg.go2flare.xyz/img/image-20220315013251109.png)



而是用alpine的大小，比scratch大不了几m



![img](http://myimg.go2flare.xyz/img/image-20220315022246982.png)



## web应用无法访问



一般docker运行时所映射端口，如果对应上网卡，将宿主机所有请求打到docker0，容器中所运行的应用如果只监听localhost网卡，则宿主机的请求应该是打不到容器对应的应用，可能会出现以下错误



```plain
connect reset by peer
```



我们在容器的应用应该改为，监听0.0.0.0



即接收所有网卡的请求



更改后问题解除++



## docker desktop无法登录镜像仓库

出现以下问题

```plain
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Error response from daemon: Get "https://10.29.3.30:30004/v2/": x509: cannot validate certificate for 10.29.3.30 because it doesn't contain any IP SANs
```

在docker desktop中配置的engine中加入以下命令

```plain
"insecure-registries": ["10.29.3.30:30004"]
```

## gitlab runner构建镜像时无法pull子容器镜像

![image-20220610174820012](http://myimg.go2flare.xyz/img/image-20220610174820012.png)

将所需容器的镜像手动推到runner所在服务器

> 其他可以拉取镜像的服务器：10.29.3.30
>
> middleware：10.29.0.122

```
# 保存镜像
docker save -o /root/deployment/multi_demo_test/golang_1.17.tar root@10.29.0.122:/usr/pro_repo/golang_1.17.tar 

# 传输镜像
scp golang_1.17.tar root@10.29.0.122:/usr/pro_repo/golang_1.17.tar

# 加载镜像
docker load -i golang_1.17.tar
```

