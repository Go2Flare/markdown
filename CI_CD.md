# 安装gitlab-runner

## gitlab-ci

### 安装runner

yum[官方](https://docs.gitlab.com/runner/install/linux-repository.html#installing-gitlab-runner)

```
Remove the old repository:
sudo rm /etc/yum.repos.d/runner_gitlab-ci-multi-runner.repo

curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash

sudo yum install gitlab-runner

For RHEL/CentOS/Fedora, run:
sudo /usr/share/gitlab-runner/post-install

Updating GitLab Runner:
sudo yum update
sudo yum install gitlab-runner
```

[官网](https://docs.gitlab.com/runner/install/linux-manually.html)

[安装gitlab-runner](https://www.jianshu.com/p/c78f8cd78d71)

```
curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/rpm/gitlab-runner_amd64.rpm"

rpm -i gitlab-runner_amd64.rpm
```

## 注册runner

gitlab可以直接在该主机上做部署

```shell
[root@10-23-12-115 ~]# gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=135323 revision=565b6c0b version=14.8.0
Running in system-mode.                            
                                                   
Enter the GitLab instance URL (for example, https://gitlab.com/):
https://gitlab.daocloud.cn/
Enter the registration token:
JH64f7yj_9tMdrZ5QowX
Enter a description for the runner:
[10-23-12-115]: dsp-alertcenter
Enter tags for the runner (comma-separated):
alertcenter
Enter optional maintenance note for the runner:
run by shell
Registering runner... succeeded                     runner=JH64f7yj
Enter an executor: docker-ssh, shell, virtualbox, docker+machine, custom, docker, parallels, ssh, docker-ssh+machine, kubernetes:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 

```

![image-20220310113520232](http://myimg.go2flare.xyz/img/image-20220310113520232.png)



# 流程

![image-20220310150124363](http://myimg.go2flare.xyz/img/image-20220310150124363.png)

## dockerfile

```dockerfile
FROM daocloud.io/atsctoo/golang:1.15 AS build-env
ENV GO111MODULE on
ENV GOFLAGS -mod=vendor
ENV GOPROXY https://goproxy.cn
ARG appRoot=/go/src/github.com/daocloud/dsp-alertcenter
ADD . $appRoot
# 增加设置 版本信息, 默认根据git自动执行, 想要自定义的话, 可以将下面命令中的 $(git xxx) 替换为自定义的值, 不设置的话为unknown
RUN cd $appRoot && go build -ldflags "-X 'github.com/daocloud/dsp-alertcenter/pkg/info.DspVersion=$(git describe --abbrev=0 --tags)' -X 'github.com/daocloud/dsp-alertcenter/pkg/info.GitCommitID=$(git rev-parse --short HEAD)'" -o /go/alertcenter ./cmd/alertcenter
FROM daocloud.io/atsctoo/rhel8-go-toolset:1.11.6-22
COPY --from=build-env /go/alertcenter /alertcenter
RUN chmod -R g+w /opt/app-root

ENV TZ Asia/Shanghai

EXPOSE 9898
CMD ["/alertcenter"]

```

### 编写.gitlab-ci.yml

```yml
variables:
  CACHE_DIR: "target/"
  # 配置
  DOCKER_USER: "admin"
  DOCKER_PW: "daocloud"
  DEPLOY_REGISTRY: "10.7.253.201" #镜像仓库
  DEPLOY_SERVER: "root@10.23.12.108" #部署的服务器
  NAME_SPACE: "rftest"
  APP_NAME: "alert-center"

  # 构建镜像
  DOCKER_FILE_PATH: "./dockerfile"
  IMAGE_NAME: "${DEPLOY_REGISTRY}/${NAME_SPACE}/$APP_NAME:${CI_COMMIT_SHORT_SHA}"
  BUILD_IMAGE: "docker build --pull -t $IMAGE_NAME ."
  PUSH_IMAGE: "docker push $IMAGE_NAME"
  REMOVE_IMAGE: "docker rmi $IMAGE_NAME"

  #部署镜像
  DEPLOYMENT_YAML_PATH: "/root/deployment/alertcenter"

cache:
  paths:
    - ${CACHE_DIR}

# 设置状态
stages:
  - build_image
  - push_image
  - deploy_image

#------------------stages-----------------#
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

#------------------jobs-----------------#
.build_image:
  stage: build_image
  script:
    - docker info
    - docker login -u ${DOCKER_USER} -p ${DOCKER_PW} ${DEPLOY_REGISTRY} #登录仓库
    - docker build -t ${IMAGE_NAME} -f ${DOCKER_FILE_PATH} . #构建镜像
    - docker images

.push_image:
  stage: push_image
  script:
    # 上传镜像到仓库
    - docker push ${IMAGE_NAME}
    - docker rmi ${IMAGE_NAME}
    - docker images

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

## 部署k8s的yaml

deployment.yaml

```yaml

```



# 部署并行job

## 设置并发数

```
vim /etc/gitlab-runner/config.toml
第一行为concurrent数量，默认是1，修改成需要的数字保存，
然后

gitlab-runner restart
```



```yaml
image: alpine

stages:
  - compile
  - test
  - package

compile:
  stage: compile
  script: cat file1.txt file2.txt > compiled.txt
  artifacts:
    paths:
    - compiled.txt
    expire_in: 20 minutes

test:
  stage: test
  script: cat compiled.txt | grep -q 'Hello world'

pack-gz:
  stage: package
  script: cat compiled.txt | gzip > packaged.gz
  artifacts:
    paths:
    - packaged.gz

pack-iso:
  stage: package
  before_script:
  - echo "ipv6" >> /etc/modules
  - apk update
  - apk add xorriso
  script:
  - mkisofs -o ./packaged.iso ./compiled.txt
  artifacts:
    paths:
    - packaged.iso
```



# 常见问题

## 流水线没开启可执行untag文件

```
This job is stuck because the project doesn't have any runners online assign
```

![image-20220310174649076](http://myimg.go2flare.xyz/img/image-20220310174649076.png)

![image-20220310174627935](http://myimg.go2flare.xyz/img/image-20220310174627935.png)

## 无法构建镜像

```
$ docker build -t ${IMAGE_NAME} -f ${DOCKER_FILE_PATH} .
unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat /home/gitlab-runner/builds/a1xpgeuv/0/sou-gz/dsp-alertcenter/dockerfile: no such file or directory
```

一般是dockerfile名称大小写出错如：Dockerfile却使用dockerfile

![image-20220310224359896](http://myimg.go2flare.xyz/img/image-20220310224359896.png)

## 无法使用fetch-pack命令

```
Fetching changes with git depth set to 20...
Reinitialized existing Git repository in /home/gitlab-runner/builds/Er2F3cez/0/Go2Flare/yaozhaofang/.git/
fatal: git fetch-pack: expected shallow list
fatal: The remote end hung up unexpectedly
```

git版本太低，一定要找官方的升级git的方法

```
sudo yum -y install https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
sudo yum install git
```

