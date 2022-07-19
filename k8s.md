# 基础命令

```
1.获取节点和服务版本信息，并查看附加信息 kubectl get nodes -o wide
2.获取指定名称空间的pod kubectl get pod -n kube-system
3.查看pod的详细信息，以yaml格式或json格式显示
kubectl get pods -o yaml kubectl get pods -o json
4.查看pod的标签信息 kubectl get pod -A --show-labels
5.根据Selector（label query）来查询pod kubectl get pod -A --selector="k8s-app=kube-dns"
6.# 查看运行pod的环境变量 kubectl exec podName env
7.# 查看指定pod的日志 kubectl logs -f --tail 500 -n kube-system kube-apiserver-k8s-master
8.# 查看所有名称空间的service信息 kubectl get svc -A
9.# 查看componentstatuses信息 kubectl get cs
10.# 查看所有configmaps信息 kubectl get cm -A
11.# 查看所有serviceaccounts信息 kubectl get sa -A
12.# 查看所有daemonsets信息 kubectl get ds -A
13.# 查看所有deployments信息 kubectl get deploy -A
14.# 查看所有replicasets信息 kubectl get rs -A
15.# 查看所有statefulsets信息 kubectl get sts -A
16.# 查看所有jobs信息 kubectl get jobs -A
17.# 查看所有ingresses信息 kubectl get ing -A
18.# 查看有哪些名称空间 kubectl get ns
19.# 查看pod的描述信息 kubectl describe pod -n kube-system kube-apiserver-k8s-master
20.# 查看指定名称空间中指定deploy的描述信息 kubectl describe deploy -n kube-system coredns
21.# 查看node或pod的资源使用情况 kubectl top node 或 kubectl top pod
22.# 查看集群信息 kubectl cluster-info 或 kubectl cluster-info dump
23.# 查看各组件信息【172.16.1.110为master机器】
kubectl -s https://172.16.1.110:6443 get componentstatuses
24.#在不进入pod中执行bash命令 kubectl exec pod pod名 -n 命名空间 -- "ps -ef"
kubectl exec -it nginx-56b8c64cb4-t97vb -- /bin/bash #进入容器
kubectl exec -it redis-9f9dc5c8b-hnqgm -n gitlab -- sh #进入容器

kubectl get pods --selector name=redis #按selector名来查找pod
kubectl get pods -o wide #查看pods所在的运行节点
kubectl get pods -o yaml #查看pods定义的详细信息
kubectl get nodes –l zone #获取zone的节点
kubectl get service serviceName -o yaml > backup.yaml #导出yaml配置文件
kubectl logs otel-collector-6f8c7c9754-wwmjz -n dsp > otel-collecor-log.txt  #导出log文件

k logs otel-collector-6f8c7c9754-5zmjj -n dsp --tail=100 # 限制100行log

kubectl get nodes --show-labels #查看现有所有标签
kubectl get pods --show-labels　　#查看pod所有标签信息
kubectl get pods -l app　　#过滤包含app的标签
kubectl get pods -L app    #过滤包含app的标签及显示值
kubectl label pods pod-demo release=canary　　#给pod-demo增加标签
kubectl label pods pod-demo release=stable --overwrite　　#修改标签

k label nodes node03 gitlab/key= #打标签
k label nodes node03 gitlab/middleware- #后面的减号表示将该标签删除

kubectl patch pvc <pvc> -p '{"metadata":{"finalizers":null}}' -n gitlab #强制删除pvc
kubectl delete pod [pod name] --force --grace-period=0 -n [namespace] #强制删除pod
kubectl delete pods <pod> --grace-period=0 --force
```

# 概念

## K8S概念

### K8S是什么

Kubernetes是容器集群管理系统，是一个开源的平台，可以实现容器集群的自动化部署、自动扩缩容、维护等功能。

通过Kubernetes你可以：

- 快速部署应用
- 快速扩展应用
- 无缝对接新的应用功能
- 节省资源，优化硬件资源的使用

我们的目标是促进完善组件和工具的生态系统，以减轻应用程序在公有云或私有云中运行的负担

## [label详解](https://blog.csdn.net/gongxiaoyi9511/article/details/124427796?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-124427796-blog-114679210.pc_relevant_antiscanv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-124427796-blog-114679210.pc_relevant_antiscanv3&utm_relevant_index=2)

### 标签是什么

标签是k8s特色的管理方式，便于**分类管理资源对象**。

一个**标签可以对应多个资源**，一个**资源也可以有多个标签**，它们是**多对多**的关系。 

一个资源拥有多个标签，可以实现不同[维度](https://so.csdn.net/so/search?q=维度&spm=1001.2101.3001.7020)的管理。

可以使用标签[选择器(selector)](https://so.csdn.net/so/search?q=选择器&spm=1001.2101.3001.7020)来指定能使用哪些标签。

当相同类型的资源越来越多，对资源划分管理是很有必要，此时就可以使用Label为资源对象 命名，以便于配置，部署等管理工作，提升资源的管理效率。label 作用类似Java包能对不同文件分开管理，让整体更加有条理，有利于维护

### label使用

#### 语法

##### 标签键：

- 一般情况下，键包括前缀和名称，用/分隔

- 前缀可以省略，省略则视为用户私有标签。
- 指定前缀，前缀必须是DNS子域，
- 名称必须，不超过63个字符
- k8s.io/和kubernetes.io/前缀是为k8s核心组件保留的

##### 标签值:

- 不超过63个字符，可以为空
- 字母数字开头和结尾
- 可以包含 - _ . 字母 数字

##### 其他:

- 每个对象都可以有多个标签，但是同一对象每个标签的键值必须是唯一的。
- 不同对象间的标签可以相同

##### 标签选择运算符

```
=:  相等
==: 相等
!=： 不相等，包括键不存在的情况
in: 在范围之内
notin:  不在这个范围之内，包含不存在这个标签的对象
exists: 

# 示例
env! = test   获取全部env的值不为 test 和不存在 env 标签的对象
```

- , （逗号）表示与的关系
  标签选择器为空，则选择所有。为null，全部不选择
- label的对应增删改查



增加节点的标签信息，这里就增加了一个标签


多维度标签，就是给节点增加多个标签用于不同的场景


查看node的lable标签

显示节点的应用标签

查找region=huanan的节点
修改标签

取消一个标签
总之:标签是为了更好的进行资源对象的相关选择与匹配

## annotation详解

Annotation，顾名思义，就是注解。Annotation可以将Kubernetes资源对象关联到任意的非标识性元数据。使用客户端（如工具和库）可以检索到这些元数据。Annotation与Label类似，也使用key/value键值对的形式进行定义。不同的Label具有严格的命名规则，它定义的是Kubernetes对象的元数据（Metadata），并且用于Label Selector。而Annotation则是用户任意定义的“附加”信息，以便于外部工具进行查找，很多时候，Kubernetes的模块自身会通过Annotation的方式标记资源对象的特殊信息。

## 关联元数据到对象

Label和Annotation都可以将元数据关联到Kubernetes资源对象。Label主要用于选择对象，可以挑选出满足特定条件的对象。相比之下，annotation 不能用于标识及选择对象。annotation中的元数据可多可少，可以是结构化的或非结构化的，也可以包含label中不允许出现的字符。annotation和label一样都是key/value键值对映射结构：



```bash
"annotations": {
  "key1" : "value1",
  "key2" : "value2"
}
```

通常来说，用Annotation来记录的信息如下。

- build信息、release信息、Docker镜像信息等，例如时间戳、release id号、PR号、镜像hash值、docker registry地址等。
- 日志库、监控库、分析库等资源库的地址信息。
- 程序调试工具信息，例如工具、版本号等。
- 团队等联系信息，例如电话号码、负责人名称、网址等。

#### 示例



```cpp
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: istio-manager
spec:
  replicas: 1
  template:
    metadata:
      annotations:
        alpha.istio.io/sidecar: ignore
      labels:
        istio: manager
    spec:
      serviceAccountName: istio-manager-service-account
      containers:
      - name: discovery
        image: harbor-001.jimmysong.io/library/manager:0.1.5
        imagePullPolicy: Always
        args: ["discovery", "-v", "2"]
        ports:
        - containerPort: 8080
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
      - name: apiserver
        image: harbor-001.jimmysong.io/library/manager:0.1.5
        imagePullPolicy: Always
        args: ["apiserver", "-v", "2"]
        ports:
        - containerPort: 8081
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
```

alpha.istio.io/sidecar 注解就是用来控制是否自动向 pod 中注入 sidecar 的。

## 参考：

[https://www.orchome.com/1342](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.orchome.com%2F1342)
 [https://jimmysong.io/kubernetes-handbook/concepts/annotation.html](https://links.jianshu.com/go?to=https%3A%2F%2Fjimmysong.io%2Fkubernetes-handbook%2Fconcepts%2Fannotation.html)

## Pod详解

本章主要介绍Pod资源的各种配置（yaml文件）和原理

### Pod介绍

![img](https://img2020.cnblogs.com/blog/1287132/202103/1287132-20210310151128350-1735966618.png)

如上图所示，每个Pod中都可以包含一个或多个Container，这些Containers 可以分为2类：

1. 用户程序所在的Container，数量可多可少
2. Pause容器，这是每个Pod都会有的一个根容器，它的作用有2个：
   - 可以以它为依据，评估整个Pod的健康状态
   - 可以在跟容器上设置IP地址，其他容器都可以使用此IP（Pod IP）实现Pod内部的网络通信。这里Pod内部通讯是指完全的Pod内部，对于不同Pod之间的通讯则采用的是虚拟二层网络技术实现，例如Flannel

 

### Pod 定义

可以通过kubectl explain命令查看每种资源的可配置项：

```
$ kubectl explain pod
KIND:     Pod
VERSION:  v1

FIELDS:
   apiVersion   <string>
   kind <string>
   metadata     <Object>
   spec <Object>
   status       <Object>

# 查看属性的子属性
$ kubectl explain pod.metadata
KIND:     Pod
VERSION:  v1

RESOURCE: metadata <Object>

FIELDS:
   annotations  <map[string]string>
   clusterName  <string>
   …
```

 

在Kubernetes中基本所有资源的一级属性都是一样的，主要包含5部分：

- apiVersion <string>：版本，由kubernetes内部定义，版本号必须能在kubectl api-versions查询到

- kind <string>：类型，由kubernetes内部定义，版本号必须可以用kubectl api-resources 查询到

- metadata <Object> ：元数据，主要是资源标识和说明，常用的有name、namespace、labels等

- spec <Object> ：描述，这是配置中最重要的一部分，里面是对各种资源配置的详细描述

- status <Object>：状态信息，里面的内容不需要定义，由kubernetes自动生成

 

在上面的属性中，spec是接下来研究的重点对象，它常见的子属性有：

- containers <[]Object>：容器列表，用于定义容器的详细信息

- nodeName <String>：根据nodeName的值将pod调度到指定Node节点上

- nodeSelector <map[]>：根据NodeSelector中定义的信息选择将该Pod调度到包含这些label的Node上

- hostNetwork <boolean>：是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络。默认用的是Pod IP。若是主机网络模式下，若是有多个Pod均用同一端口的话，则会有冲突，所以很少改动此配置

- volumes <[]Object>：存储卷，用于定义Pod上面挂载的存储信息

- restartPolicy <String>：重启策略，表示Pod在遇到故障时的处理策略

 

### Pod 配置

本小节主要研究 pod.spec.containers 属性，这也是pod属性中最关键的一项配置

```
$ kubectl explain pod.spec.containers
KIND:     Pod
VERSION:  v1
RESOURCE: containers <[]Object>
FIELDS:
   args <[]string>                                 # 容器的启动命令需要的参数列表
   command      <[]string>                 # 容器的启动命令列表，如不指定，使用打包时的启动命令
   env  <[]Object>                               # 容器环境变量的配置
   image        <string>                         # 容器需要的镜像地址
   imagePullPolicy      <string>          # 镜像拉取策略
   name <string> -required-             # 容器名称
   ports        <[]Object>                     # 容器需要暴露的端口号列表
   resources    <Object>                    # 资源限制和资源请求的设置
```

 

#### 基本配置

创建 pod-base.yaml 文件，内容为：

```
$ cat yamls/pod-base.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-base
  namespace: dev
  labels:
     user: zack
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  - name: busybox                  
    image: busybox:1.30
```

上面定义了1个简单的Pod，里面有2个containers：

1. nginx：用1.17.1版本的nginx创建镜像
2. busybox：用1.30版本的busybox镜像创建（busybox是一个小巧的linux命令集合）

 

```
# 创建pod
$ kubectl apply -f yamls/pod-base.yaml
pod/pod-base created

# 查看pod状态
$ kubectl get pod -n dev
NAME       READY   STATUS             RESTARTS   AGE
pod-base   1/2     CrashLoopBackOff   1          12s

=> 这里ready 1/2 表示，一共有2个containers，当前已经ready的只有1个
=> restarts 表示重启了 1 次

# 再次查看pod状态
$ kubectl get pod -n dev
NAME       READY   STATUS     RESTARTS   AGE
pod-base   1/2     NotReady   3          58s

=> 可以看到restart 有3 次，说明重启次数增加，且pod状态为NotReady，说明 pod 启动出现问题

# 使用describe 查看详细状态
$ kubectl describe pod pod-base -n dev

Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  5m8s                 default-scheduler  Successfully assigned dev/pod-base to ip-10-0-1-217.cn-north-1.compute.internal
  Normal   Pulled     5m7s                 kubelet            Container image "nginx:1.17.1" already present on machine
  Normal   Created    5m7s                 kubelet            Created container nginx
  Normal   Started    5m6s                 kubelet            Started container nginx
  Normal   Pulling    5m6s                 kubelet            Pulling image "busybox:1.30"
  Normal   Pulled     5m                   kubelet            Successfully pulled image "busybox:1.30"
  Normal   Started    4m21s (x4 over 5m)   kubelet            Started container busybox
  Normal   Created    3m39s (x5 over 5m)   kubelet            Created container busybox
  Normal   Pulled     3m39s (x4 over 5m)   kubelet            Container image "busybox:1.30" already present on machine
  Warning  BackOff    3s (x24 over 4m59s)  kubelet            Back-off restarting failed container

=> 可以看到nginx 是没问题的，但是busybox一直在尝试重启。此问题是测试环境中故意制造的问题，为了方便展示pod的配置与启动。
```



#### 镜像拉取

imagePullPolicy 用于设置镜像拉取策略，kubernetes支持配置3种拉取策略：

- Always：总是从远程拉取镜像
- IfNotPresent：本地有则使用本地镜像，本地没有则从远程仓库拉取镜像
- Never：只是用本地镜像，从不去远程仓库拉取，本地没有就报错

默认值说明：

- 如果镜像tag为具体版本号，默认策略是：IfNotPresent
- 如果镜像tag为：latest，默认策略是：Always

 

一个示例Pod配置文件：

```
$ cat yamls/pod-base.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-base
  namespace: dev
  labels:
     user: zack
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    imagePullPolicy: Always
  - name: busybox
image: busybox:1.30
```

 

#### Pod启动命令

在之前的例子中，busybox的pod会一直重试并失败。这是因为busybox并不是一个程序，而是类似于一个工具的集合，kubernetes集群启动管理后，它会自动关闭。解决方法就是让它一直处于运行状态，这就用到了command配置。

创建pod-command.yaml 文件，内容如下：

```
$ cat yamls/pod-command.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-command
  namespace: dev
  labels:
     user: zack
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    imagePullPolicy: Always
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh", "-c", "touch /tmp/hello.txt;while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;"]
```

 

运行并检查：

```
$ kubectl apply -f yamls/pod-command.yaml
pod/pod-command created

$ kubectl get pods -n dev
NAME          READY   STATUS             RESTARTS   AGE
pod-base      1/2     CrashLoopBackOff   8          20m
pod-command   2/2     Running            0          6s
```



可以看到pod-command 中2个container均是running 状态。

在busybox中，我们运行的command 内容是将日期输入到一个临时文件中，若是需要进入container检查此文件，可以执行：

```
# 进入pod中的busybox container 查看文件内容
# 命令格式：kubectl exec <pod_name> -n <namespace> -it -c <container_name> -- <command>

$ kubectl exec pod-command -n dev -it -c busybox -- /bin/sh
/ # tail /tmp/hello.txt
07:19:40
07:19:43
07:19:46
07:19:49
07:19:52
07:19:55
07:19:58
07:20:01
07:20:04
07:20:07

/ # exit
```

 

从这个例子中可以看到，command的功能可以完成启动命令和传递参数的功能，但是pod中仍提供了一个args选项用于传递参数。这点和docker有些关系，kubernetes中的command、args两项其实是实现覆盖Dockerfile中的ENTRYPOINT的功能：

1. 如果command和args均没有写，则用Dockerfile的配置
2. 如果command写了，但args没有写，则Dockerfile的默认配置会被忽略，执行输入的command
3. 如果command没写，但args写了，则Dockerfile中配置的ENTRYPOINT的命令会被执行，使用当前的args参数
4. 如果command和args都写了，则Dockerfile的配置被忽略，执行command并追加上args参数

 

####  环境变量

env主要用于传递container环境变量，主要格式是key-value 方式。生产环境使用较少，也并不推荐使用这种方式配置环境变量。下面仅举例：

创建pod-env.yaml：

```
$ cat yamls/pod-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-command
  namespace: dev
  labels:
     user: zack
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh", "-c", "touch /tmp/hello.txt;while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;"]
    env:
    - name: "username"
      value: "admin"
    - name: "password"
      value: "123"
```

 

部署后验证：

```
$ kubectl exec pod-command -it -n dev -- /bin/sh
/ # echo $username
admin
/ # echo $password
123
/ #
```

此方式配置环境并非推荐方式，推荐将这些配置单独存储在配置文件中。

 

#### 端口设置

ports支持的子选项：

```
$ kubectl explain pod.spec.containers.ports
KIND:     Pod
VERSION:  v1

RESOURCE: ports <[]Object>

FIELDS:
   containerPort        <integer>         # 容器要监听的端口（0 < x < 65536）
   hostIP       <string>         # 要将外部端口绑定到的主机IP（一般省略） 
   hostPort     <integer>    # 容器要在主机上公开的端口，如果设置，主机上只能运行容器的一个副本（一般省略）
   name <string>                            # 端口名称，如果指定，必须保证name在pod中是唯一的
   protocol     <string>        # 端口协议。必须是UDP、TCP或SCTP，默认为TCP
```

 

创建 pod-ports.yaml：

```
$ cat yamls/pod-ports.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-ports
  namespace: dev
  labels:
     user: zack
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    imagePullPolicy: Always
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```

```
$ kubectl apply -f yamls/pod-ports.yaml
pod/pod-ports created
```

这里是定义containerPort，要访问此服务，需要通过 Pod IP + Container 端口。

 

#### 资源配额

容器中的程序要运行，肯定是要占用一定的资源，比如cpu和内存等。如果不对某个容器的资源做限制，那么它可能会用掉大量资源，而导致其他containers无法运行。对于这种情况，kubernetes 提供了对内存和cpu的资源进行配额的机制。这种机制主要通过resources选项实现，有2个子选项：

1. limits：用于限制运行时容器的最大占用资源，当容器占用资源超过limits时会被终止，并进行重启
2. requests：用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动

 

可以通过这两个选项分别设置资源的上下限。

如下pod-resources.yaml 所示的例子：

```
$ cat yamls/pod-resources.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-resources
   namespace: dev
spec:
   containers:
   - name: nginx
     image: nginx:1.17.1
     resources:
        limits:
           cpu: "2"
           memory: "10Gi"
        requests:
           cpu: "1"
           memory: "10Mi"
```

这里对cpu和memory的单位做一个说明：

1. cpu：core数，可以为整数或是小数
2. memory：内存大小，可以使用Gi、Mi、G或M等形式

 

### Pod 生命周期

我们将pod对象从创建至终的这段时间范围称为pod的生命周期，它主要包含以下过程：

1. Pod创建过程
2. 运行初始化容器（init container）过程
3. 运行主容器（mian container）过程
   - 容器启动后钩子（post start）、容器终止前钩子（pre stop）
   - 容器的存活性探测（liveness probe）、就绪性探测（readiness probe）
4. Pod终止过程

如下所示：

![img](https://img2020.cnblogs.com/blog/1287132/202103/1287132-20210310152328755-2060487254.png)

在整个生命周期中，pod会出现5种状态（相位）：

1. 挂起（Pending）：apiserver已经创建了pod资源对象，但它尚未被调度完成或者仍处于下载镜像的过程中
2. 运行中（Running）：pod已经被调度至某节点，并且所有容器都已经被kubelet创建完成
3. 成功（Succeeded）：pod中的所有容器都已经成功终止，且不会被容器
4. 失败（Failed）：所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的退出状态
5. 未知（Unknown）：apiServer无法正常获取到pod对象的状态信息，通常由网络通信失败导致

 

#### Pod的创建与终止

**Pod**创建过程：

1. 用户通过kubectl 或其他api客户端提交需要创建的pod信息给apiServer
2. apiServer开始生成pod对象的信息，并将信息存入etcd，然后返回确认信息至客户端
3. apiServer开始反映etcd中的pod对象的变化，其他组件使用watch机制来跟踪检查apiServer上的变动
4. scheduler发现有新的pod对象要创建，开始为pod分配主机并将结果信息更新至apiServer
5. node节点上的kubelet发现有pod调度过来，尝试调用docker启动容器，并将结果回送至apiServer
6. apiServer将接收到的pod状态信息存入etcd中

![img](https://img2020.cnblogs.com/blog/1287132/202103/1287132-20210310152640802-1826265419.png)

整个过程中基本上是各个组件监听ApiServer的变动来触发自身的工作。例如scheduler 发现有新的pod要创建后，会分配主机并将结果信息更新给ApiServer。此时kubelet由于在watch ApiServer，所以会感知到这个变动，继而开始启动container，并将结果返回给ApiServer。

**Pod**的终止过程：

1. 用户向apiServer发送删除pod对象的命令
2. apiServer中的pod对象信息会随着时间的推移而更新，在宽限期内（默认30s），pod被视为dead
3. 将pod标记为terminating状态
4. kubelet 在监控到pod对象转为terminating状态的同时启动pod关闭过程
5. 端点控制器监控到pod对象的关闭行为时将其从所有匹配到此端点的service资源的端点列表中移除
6. 如果当前pod对象中定义了preStop钩子，则在其标记为terminating后即会以同步的方式启动执行
7. Pod对象中的容器进程收到停止信号
8. 宽限期结束后，若pod中还存在仍在运行的进程，那么pod对象会收到立即终止的信号
9. kubelet请求apiServer将此pod资源的宽限期设置为0从而完成删除操作，此时pod对于用户已不可见

过程如下图所示：

![img](https://img2020.cnblogs.com/blog/1287132/202103/1287132-20210310152839510-311482717.png)

#### 初始化容器

初始化容器是在pod的主容器启动之前要运行的容器，主要是做一些容器的前置工作，具有2大特征：

1. 初始化容器必须运行完成至结束，若某初始化容器运行失败，那么kubernetes需要重启它直到成功完成
2. 初始化容器必须按照定义的顺序执行，当且仅当一个成功后，后面的一个才能运行

初始化容器有很多的应用场景，常见的有：

1. 提供主容器镜像中不具备的工具程序或自定义代码
2. 初始化容器要先于应用容器串行启动并运行完成，因此可用于延后应用容器的启动直至其依赖的条件得到满足

假设有这么一个需求：要以主容器来运行nginx，但是要求在运行nginx之前要能够连接上mysql和redis所在的服务器。为了简化测试，ping的目标服务的地址为测试地址。

创建pod-initcontainer.yaml：

```
$ cat yamls/pod-initcontainer.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-initcontainer
   namespace: dev
spec:
   containers:
   - name: main-container
     image: nginx:1.17.1
     ports:
     - name: nginx-port
       containerPort: 80
   initContainers:
   - name: test-mysql
     image: busybox:1.30
     command: ['/bin/sh', '-c', 'until ping 10.0.1.217 -c 1 ; do echo waiting for mysql...; sleep 2; done;']
   - name: test-redis
     image: busybox:1.30
     command: ['/bin/sh', '-c', 'until ping 10.0.1.217 -c 1 ; do echo waiting for mysql...; sleep 2; done;']
```

使用kubectl get pods -n dev -o wide -w 命令进行持续监控（-w 参数进行监控）：

```
$ kubectl get pods -n dev -o wide -w
NAME                READY   STATUS     RESTARTS   AGE    IP          NODE                                        
nginx               1/1     Running    0          102m   10.0.1.84   ip-10-0-1-217.cn-north-1.compute.internal   
pod-initcontainer   0/1     Init:0/2   0          1s     <none>      ip-10-0-1-217.cn-north-1.compute.internal   
pod-initcontainer   0/1     Init:1/2   0          2s     10.0.1.31   ip-10-0-1-217.cn-north-1.compute.internal   
pod-initcontainer   0/1     PodInitializing   0          3s     10.0.1.31   ip-10-0-1-217.cn-north-1.compute.internal   
pod-initcontainer   1/1     Running           0          4s     10.0.1.31   ip-10-0-1-217.cn-north-1.compute.internal  
```

可以看到在Init阶段启动了2个container，然后再启动的main container

 

#### 钩子函数

钩子函数能够感知自身生命周期中的事件，并在相应的时刻到来时运行用户指定的程序代码。

Kubernetes 在main container 的启动之后和停止之前提供了2个钩子函数：

1. post start：容器创建之后执行，如果失败了会重启容器
2. pre stop：容器终止之前执行，执行完成后容器将成功终止，在其完成之前会阻塞删除容器的操作

钩子处理器支持使用以下3种方式定义动作：

1. Exec命令：在容器内执行1次命令
2. TCPSocket：在当前容器尝试访问指定socket
3. HTTPGet：在当前容器中向某url发起http请求

以exec方式举例：

```
$ cat yamls/pod-hook-exec.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-hook-exec
   namespace: dev
spec:
   containers:
   - name: main-container
     image: nginx:1.17.1
     ports:
     - name: nginx-port
       containerPort: 80
     lifecycle:
       postStart:
          exec:
            command: ["/bin/sh", "-c", "echo postStart... > /usr/share/nginx/html/index.html"]
       preStop:
          exec:
            command: ["/usr/sbin/nginx", "-s", "quit"]
```

```
# 验证：
$ curl 10.0.1.45
postStart...
```

 

#### 容器探测

容器探测用于检测容器中的应用实例是否正常工作，是保障业务可用性的一种传统机制。如果经过探测，实例的状态不符合预期，那么kubernetes就会把问题实例“摘除”，不承担业务流量。Kubernetes提供了2种探针来实现容器探测，分别是：

1. liveness probes：存活性探针，用于检测应用实例当前是否处于正常运行状态，如果不是k8s会重启容器
2. readiness probes：就绪性探针，用于检测应用实例当前是否可以接收请求，如果不能，ks不会转发流量

livenessProbe决定是否重启容器，readinessProbe决定是否将请求转发给容器

上面2种探针均支持3种探测方式：

1. Exec命令：在容器内执行1次命令，如果命令执行的退出码为0，则认为程序正常，否则不正常
2. TCPSocket：将会尝试访问一个用户容器的端口，如果能够建立这条连接，则认为程序正常，否则不正常
3. HTTPGet：调用容器内Web应用的URL，如果返回的状态码在200和399之间，则认为程序正常，否则不正常

以 liveness probe为例：

Exec方式：

```
$ cat yamls/pod-liveness-exec.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-liveness-exec
   namespace: dev
spec:
   containers:
   - name: main-container
     image: nginx:1.17.1
     ports:
     - name: nginx-port
       containerPort: 80
     livenessProbe:
       exec:
         command: ["/bin/cat", "/tmp/hello.txt"]
```

 

检查：

```
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  70s               default-scheduler  Successfully assigned dev/pod-liveness-exec to ip-10-0-1-217.cn-north-1.compute.internal
  Normal   Pulled     9s (x3 over 69s)  kubelet            Container image "nginx:1.17.1" already present on machine
  Normal   Created    9s (x3 over 69s)  kubelet            Created container main-container
  Normal   Started    9s (x3 over 69s)  kubelet            Started container main-container
  Warning  Unhealthy  9s (x6 over 59s)  kubelet            Liveness probe failed: /bin/cat: /tmp/hello.txt: No such file or directory
  Normal   Killing    9s (x2 over 39s)  kubelet            Container main-container failed liveness probe, will be restarted
```

 

可以看到main-container 的liveness probe 检测失败，会自动重启。也可以通过kubectl get 进行检查：

```
$ kubectl get pods -n dev -w
NAME                READY   STATUS    RESTARTS   AGE
pod-liveness-exec   1/1     Running   4          2m15s
pod-liveness-exec   0/1     CrashLoopBackOff   4          2m32s
pod-liveness-exec   1/1     Running            5          3m13s
pod-liveness-exec   0/1     CrashLoopBackOff   5          3m41s
…
```

 

livenessProbe 的子属性还有其他配置项，如下所示：

```
$ kubectl explain pod.spec.containers.livenessProbe
KIND:     Pod
VERSION:  v1

RESOURCE: livenessProbe <Object>

FIELDS:
   exec <Object>
   failureThreshold     <integer>           # 连续探测失败多少次才被认定为失败。默认为3，最小为1
   httpGet      <Object>
   initialDelaySeconds  <integer>         # 容器启动后等待多少秒执行第1次探测
   periodSeconds        <integer>           # 执行探测的频率。默认是10秒，最小1秒
   successThreshold     <integer>         # 连续探测多少次才被认定成功。默认为3，最小为1
   tcpSocket    <Object>                            
   timeoutSeconds       <integer>         # 探测超时时间。默认1秒，最小1秒
```

 

#### 重启策略

在探测容器出现问题时，kubernetes会对容器所在的Pod进行重启，这是由pod的重启策略决定的，pod的重启策略有3种，分别为：

1. Always：容器失效时，自动重启该容器；默认值
2. OnFailure：容器终止运行且退出码不为0时重启
3. Never：无论状态为何，都不重启该容器

重启策略适用于pod对象中所有的容器，首次需要重启的容器，将在其需要时立即进行重启，随后再次需要重启的操作将由kubelet延迟一段时间后进行。且反复重启操作的延时以10s、20s、40s、80s、160s和300s 进行递增。300s为最大延迟时长。

### Pod 调度

默认情况下，一个Pod在哪个Node上运行，是由Scheduler组件采用相应的算法计算出来的，这个过程不受人工控制。不过在实际使用中，仍有需要控制Pod调度到某些节点。Kubernetes提供了4大类调度方式：

1. 自动调度：运行在哪个节点上完全由Scheduler经过一系列的算法计算得出
2. 定向调度：NodeName、NodeSelector
3. 亲和性调度：NodeAffinity、PodAffinity、PodAntiAffinity
4. 污点（容忍）调度：Taints、Toleration

#### 定向调度

定向调度，是指利用Pod上声明nodeName 或是nodeSelector的方式，将Pod调度到指定节点上。此方式是强制性的，也就是说，即使Node节点不存在，也会强行调度，只是pod运行会失败。

##### NodeName

强制约束将Pod调度到指定Name的Node节点上。此方式会直接跳过Scheduler的调度逻辑，直接将Pod调度到指定名称的节点。

例如：

```
$ cat yamls/pod-nodename.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-nodename
   namespace: dev
spec:
   containers:
   - name: nginx
     image: nginx:1.17.1
   nodeName: ip-10-0-1-217.cn-north-1.compute.internal
```

```
$ kubectl get pods -n dev -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP          NODE                                        
pod-nodename   1/1     Running   0          10s   10.0.1.84   ip-10-0-1-217.cn-north-1.compute.internal   
```

 

可以看到pod被调度到此node 上。

如果强制指定一个不存在的nodeName：

```
$ kubectl get pods -n dev -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP       NODE    
pod-nodename   0/1     Pending   0          9s    <none>   node1  
```

##### NodeSelector

NodeSelector用于将pod调度到添加了指定标签的node节点上。它是通过kubernetes 的label-selection机制实现。也就是说，在pod创建之前，会由scheduler使用MatchNodeSelector调度策略进行label匹配，找到目标node进行调度。该匹配规则是强制约束。

例如：

```
# 给node打标签
$ kubectl label node ip-10-0-1-217.cn-north-1.compute.internal nodeenv=pro
node/ip-10-0-1-217.cn-north-1.compute.internal labeled

$ cat yamls/pod-nodelabel.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-nodename
   namespace: dev
spec:
   containers:
   - name: nginx
     image: nginx:1.17.1
   nodeSelector:
         nodeenv: pro

$ kubectl apply -f yamls/pod-nodelabel.yaml
pod/pod-nodename created

$ kubectl get pods -n dev -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP           NODE                                        
pod-nodename   1/1     Running   0          18s   10.0.1.216   ip-10-0-1-217.cn-north-1.compute.internal   
```

### 亲和性调度

在强制调度下，如果没有符合条件的节点，pod就会pending，无法正常运行。所以kubernetes还提供了一种亲和性调度（Affinity）。它在NodeSelector的基础上进行了扩展，可以通过配置的方式，实现优先选择满足条件的Node进行调度。如果没有合适节点，则也可以调度到不满足条件的节点上，使调度更加灵活。

Affinity主要分为3类：

1. **nodeAffinity（node亲和性）**：以node为目标，解决pod可以调度到哪些node的问题
2. **podAffinity（pod亲和性）**：以pod为目标，解决pod可以和哪些**已存在的pod部署在同一个拓扑域**中的问题
3. **podAntiAffinity（pod反亲和性）**：以pod为目标，解决pod**不能**和哪些**已存在pod部署在同一个拓扑域**中的问题

关于亲和性（与反亲和性）使用场景的说明：

- 亲和性：如果2个应用频繁交互，那就有必要利用亲和性让2个应用尽可能地靠近，这样可以减少因网络通信而带来的性能损耗
- 反亲和性：当应用采用**多副本部署**（高可用）时，有必要采用反亲和性让各个应用实例打散分布在各个node上，这样可以提高服务的高可用性

 

nodeAffinity亲和性配置：

```
pod.spec.affinity.nodeAffinity
     requiredDuringSchedulingIgnoredDuringExecution      # Node 节点必须满足指定所有规则才可以，相当于硬限制
            nodeSelectorTerms         节点选择列表
                    matchFields               按节点字段列出的节点选择器要求列表
                    matchExpression       按节点标签列出的节点选择器要求列表（推荐）
                           key
                           value
                           operator               关系符，支持Exists、DoesNotExists、In、NotIn、Gt、Lt

     preferredDuringSchedulingIgnoredDuringExecution     # 优先调度到满足指定规则的Node，相当于软限制（倾向）
                    preference                  一个节点选择器项，与相应的权重相关联
                        matchFields               按节点字段列出的节点选择器要求列表
                        matchExpression       按节点标签列出的节点选择器要求列表（推荐）
                             key
                             value
                             operator               关系符，支持Exists、DoesNotExists、In、NotIn、Gt、Lt
                     weight                           倾向权重，范围1-100
 
```

 

关系符的使用说明：

```
- matchExpressions:
   - key: nodeenv                       # 匹配存在标签的key为nodeenv的节点
     operator: Exitsts
   - key: nodeenv                       # 匹配标签的key为nodeenv，且value是xxx或yyy的节点
     operator: In
     values: ["xxx", "yyy"]
   - key: nodeenv                       # 匹配标签的key 为nodeenv，且value大于xxx的节点
     operator: Gt
     values: "xxx"
```

#### requiredDuringSchedulingIgnoredDuringExecution

下面演示requiredDuringSchedulingIgnoredDuringExecution

```
$ cat yamls/pod-nodeaffinity-required.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-nodeaffinity-required
   namespace: dev
spec:
   containers:
   - name: nginx
     image: nginx:1.17.1
   affinity:
     nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: nodeenv
                  operator: In
                  values: ["xxx", "yyy"]
```

 

在部署后，可以看到Pod 是pending状态。原因是：

```
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  33s (x4 over 3m28s)  default-scheduler  0/2 nodes are available: 2 node(s) didn't match node selector.
```

由于没有label符合条件，所以pod无法正常启动。使用能启动的方式：

```
#为节点打上标签
kubectl label nodes ip-10-0-1-217.cn-north-1.compute.internal nodeenv=pro

# 修改调度策略
- matchExpressions:
                - key: nodeenv
                  operator: In
                  values: ["pro", "yyy"]

# 可以正常启动
$ kubectl get pods -n dev
NAME                        READY   STATUS    RESTARTS   AGE
pod-nodeaffinity-required   1/1     Running   0          14s
```

 

#### preferredDuringSchedulingIgnoredDuringExecution

```
$ cat yamls/pod-nodeaffinity-preferred.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-nodeaffinity-preferred
   namespace: dev
spec:
   containers:
   - name: nginx
     image: nginx:1.17.1
   affinity:
     nodeAffinity:
        preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
             matchExpressions:
             - key: nodeenv
               operator: In
               values: ["xxx", "yyy"]
```

 

由于是软限制，所以pod能正常启动：

```
$ kubectl get pods -n dev
NAME                         READY   STATUS    RESTARTS   AGE
pod-nodeaffinity-preferred   1/1     Running   0          27s
```

 

NodeAffinity规则设置的注意事项：

1. 如果同时定义了nodeSelector和nodeAffinity，那么必须2个条件都得到满足，Pod才能运行在指定的Node上
2. 如果nodeAffinity指定了多个nodeSelectorTerms，则只需匹配到其中1个成功即可
3. 如果一个nodeSelectorTerms中有多个matchExpressions，则1个节点必须满足所有条件才能匹配成功
4. 如果一个pod所在的Node在Pod运行期间标签发生了改变，不再符合该Pod的节点亲和性需求，则系统将忽略此变化

 

#### PodAffinity

以正在运行的pod作为参照，决定新pod的调度。

PodAffinity的可配置项

```
$ kubectl explain pod.spec.affinity.podAffinity
KIND:     Pod
VERSION:  v1

RESOURCE: podAffinity <Object>

FIELDS:
   requiredDuringSchedulingIgnoredDuringExecution       <[]Object>    # 硬限制
       namespaces        # 指定参照pod的namespace
       topologyKey       # 指定调度作用域
       labelSelector      # 标签选择器
           matchExpressions     # 按节点标签列出节点选择器要求列表（推荐）
                key
                value
                operator     # 关系符，支持In、NotIn、Exists、DoesNotExist
            matchLabels    # 指多个matchExpression 映射的内容
 
   preferredDuringSchedulingIgnoredDuringExecution      <[]Object>   # 软限制
       namespaces        # 指定参照pod的namespace
       topologyKey       # 指定调度作用域
       labelSelector      # 标签选择器
           matchExpressions     # 按节点标签列出节点选择器要求列表（推荐）
                key
                value
                operator     # 关系符，支持In、NotIn、Exists、DoesNotExist
            matchLabels    # 指多个matchExpression 映射的内容
       weight                   # 倾向权重，范围为1-100
```

topologyKey 用于指定调度时作用域，例如：

1. 如果指定为kubernetes.io/hostname，那就是以Node节点为区分范围
2. 如果指定为beta.kubernetes.io/os 则以Node节点的操作系统类型来区分

 

演示requiredDuringSchedulingIgnoredDuringExecution

```
# 首先启动一个目标pod
$ kubectl get pods -n dev -o wide --show-labels
NAME    READY   STATUS    RESTARTS   AGE    IP           NODE                                       NOMINATED NODE   READINESS GATES   LABELS
nginx   1/1     Running   0          111s   10.0.2.186   ip-10-0-2-30.cn-north-1.compute.internal   <none>           <none>            tier=production,version=3.0
```

```
$ cat yamls/pod-podaffinity-required.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-podaffinity-required
   namespace: dev
spec:
   containers:
   - name: nginx
     image: nginx:1.17.1
   affinity:
     podAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
         - labelSelector:
               matchExpressions:
               - key: tier
                 operator: In
                 values: ["production", "yyy"]
           topologyKey: kubernetes.io/hostname
```

成功启动，可以看到调度到了与target pod同一节点：

```
$ kubectl get pods -n dev -o wide --show-labels
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE                                      LABELS
nginx                      1/1     Running   0          12m   10.0.2.186   ip-10-0-2-30.cn-north-1.compute.internal   tier=production,version=3.0
pod-podaffinity-required   1/1     Running   0          46s   10.0.2.124   ip-10-0-2-30.cn-north-1.compute.internal   <none>
```

 

**Pod**反亲和性

```
$ cat yamls/pod-podantiaffinity-required.yaml
apiVersion: v1
kind: Pod
metadata:
   name: pod-podantiaffinity-required
   namespace: dev
spec:
   containers:
   - name: nginx
     image: nginx:1.17.1
   affinity:
     podAntiAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
         - labelSelector:
               matchExpressions:
               - key: tier
                 operator: In
                 values: ["production", "yyy"]
           topologyKey: kubernetes.io/hostname
```

 

成功启动，可以看到调度到了与target node 不同的节点上：

```
$ kubectl get pods -n dev -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE                                      GATES
nginx                          1/1     Running   0          28m   10.0.2.186   ip-10-0-2-30.cn-north-1.compute.internal    
pod-podantiaffinity-required   1/1     Running   0          12s   10.0.1.216   ip-10-0-1-217.cn-north-1.compute.internal   
```

 

### 污点和容忍

#### 污点（Tiants）

可以通过在Node上添加污点属性，来决定是否允许Pod调度过来。

Node在被设置上污点之后，就和Pod之间存在了一种相斥的关系，进而拒绝Pod调度进来，甚至可以将已经存在的Pod驱逐出去。

污点的格式为：key=value:effect。Key和value是污点的标签，effect描述污点的作用，支持如下3个选项：

1. PreferNoSchedule：kubernetes将尽量避免把Pod调度到具有该污点的Node上，除非没有其他节点可调度
2. NoSchedule：kubernetes将不会把Pod调度到具有该污点的Node上，但不会影响当前Node上已存在的Pod
3. NoExecute：kubernetes将不会把Pod调度到具有该污点的Node上，同时也会将Node上已存在的Pod驱离

![img](https://img2020.cnblogs.com/blog/1287132/202103/1287132-20210310154946731-359245651.png)

设置&去除污点的常规写法：

```
# 设置污点
kubectl taint nodes node1 key=value:effect

# 去除污点
kubectl taint nodes node1 key:effect-

# 去除所有污点
kubectl taint nodes node1 key-
```

演示：

```
# 为节点设置污点：
$ kubectl taint nodes ip-10-0-2-30.cn-north-1.compute.internal tag=wever:PreferNoSchedule

# 启动pod 后的结果：
$ kubectl get pods -n dev -o wide
NAME     READY   STATUS    RESTARTS   AGE     IP           NODE                                        
nginx    1/1     Running   0          2d19h   10.0.2.186   ip-10-0-2-30.cn-north-1.compute.internal    
taint1   1/1     Running   0          15s     10.0.1.216   ip-10-0-1-217.cn-north-1.compute.internal   
taint2   1/1     Running   0          8s      10.0.1.84    ip-10-0-1-217.cn-north-1.compute.internal   

# 去除污点，并设置NoExcuete级别：
$ kubectl taint nodes ip-10-0-2-30.cn-north-1.compute.internal tag=wever:PreferNoSchedule-
node/ip-10-0-2-30.cn-north-1.compute.internal untainted

$ kubectl get pods -n dev -o wide
NAME     READY   STATUS        RESTARTS   AGE     IP           NODE                                        
nginx    0/1     Terminating   0          2d20h   10.0.2.186   ip-10-0-2-30.cn-north-1.compute.internal    
taint1   1/1     Running       0          9m27s   10.0.1.216   ip-10-0-1-217.cn-north-1.compute.internal   
taint2   1/1     Running       0          9m20s   10.0.1.84    ip-10-0-1-217.cn-north-1.compute.internal   

=> 可以看到此节点上的pod立即开始关闭
```

对于master节点，默认打了NoSchedule级别污点，所以不会调度pods在master节点上。

#### 容忍（Toleration）

在node上可以通过污点来拒绝pod调度上来。但是如果想将一个pod调度到一个有污点的node上，仍可以通过容忍的机制实现。如下图所示：

![img](https://img2020.cnblogs.com/blog/1287132/202103/1287132-20210310155032712-1279684851.png)

污点就是拒绝，容忍就是忽略它的拒绝。Node通过污点拒绝pod调度上去，Pod通过容忍忽略拒绝。

演示：

```
# 给pod打上NoExecute污点
$ kubectl taint nodes ip-10-0-2-30.cn-north-1.compute.internal tag=wever:NoExecute
node/ip-10-0-2-30.cn-north-1.compute.internal tainted

# 创建pod定义
$ cat yamls/pod-toleration.yaml
apiVersion: v1
kind: Pod
metadata:
   namespace: dev
   name: pod-toleration
spec:
   containers:
   - name: nginx
     image: nginx:1.17.1
   tolerations:                       # 添加容忍
   - key: "tag"                        # 要容忍的污点key
     operator: "Equal"          # 操作符
     value: "wever"               # 容忍的污点value
     effect: "NoExcude"       # 添加容忍的规则，这里必须和标记的污点规则相同

# 可以看到pod被正常调度到NoExecute节点
$ kubectl get pods -n dev -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP           NODE                                        
pod-toleration   1/1     Running   0          37s   10.0.2.186   ip-10-0-2-30.cn-north-1.compute.internal    
taint2           1/1     Running   0          24m   10.0.1.84    ip-10-0-1-217.cn-north-1.compute.internal   
```

## 重启pod

一、有yaml文件的重启方式
在有 yaml 文件的情况下可以直接使用

```
kubectl replace --force -f xxx.yaml 
```

来强制替换Pod 的 API 对象，从而达到重启的目的。

二、没有yaml文件的重启方式
使用scale命令
没有 yaml 文件，但是使用的是 Deployment 对象。可以使用以下方式重启

```
kubectl scale deployment esb-admin --replicas=0 -n {namespace}
kubectl scale deployment esb-admin --replicas=1 -n {namespace}
```

由于 Deployment 对象并不是直接操控的 Pod 对象，而是操控的 ReplicaSet 对象，而 ReplicaSet 对象就是由副本的数目的定义和Pod 模板组成的。所以这条命令分别是将ReplicaSet 的数量 scale 到 0，然后又 scale 到 1，那么 Pod 也就重启了。

直接删除Pod进行重启
同样没有 yaml 文件，但是使用的是 Deployment 对象。查看deploy文件的重启策略，如果配置了重启策略。可以尝试删除重启：


使用命令

```
kubectl delete pod {podname} -n {namespace}
```

这个方法就很简单粗暴了，直接把 Pod 删除，因为 Kubernetes 是声明式 API，所以删掉了之后，Pod API 对象就与预期的不一致了，所以会自动重新创建 Pod 保持与预期一致，但是如果ReplicaSet 管理的 Pod 对象很多的话，那么要一个个手动删除，会很麻烦，所以可以使用

```
kubectl delete replicaset {rs_name} -n {namespace}
```

命令来删除 ReplicaSet

使用“-o yaml”参数导出Pod模板并重建Pod【推荐】
没有 yaml 文件，直接使用的 Pod 对象。

使用命令

```
kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f -
```

在这种情况下，由于没有 yaml 文件，且启动的是 Pod 对象，那么是无法直接删除或者 scale 到 0 的，但可以通过上面这条命令重启。这条命令的意思是 get 当前运行的 pod 的 yaml声明，并管道重定向输出到 kubectl replace命令的标准输入，从而达到重启的目的。

总结
我们可以通过多种方式来重启对象，总的来说，最推荐的方式是使用

```
kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f -
```


这种方式，因为适用于多种对象。此外，重启 Pod 并不会修复运行程序的 bug，想要解决程序的意外终止，最终还是得要修复 bug。

# Job使用

## 运行一次性容器

容器按照持续运行的时间可分为两类：

**服务类容器**

服务类容器通常持续提供服务，需要一直运行，比如 http server，daemon 等。

**工作类容器**

工作类容器则是**一次性任务**，比如**批处理程序，完成后容器就退出**。

Kubernetes 的 Deployment、ReplicaSet 和 DaemonSet 都用于**管理服务类**容器；对于**工作类**容器，我们用 Job。

先看一个简单的 Job 配置文件 myjob.yml：

```
spec: 
  template: 
    metadata: 
      name: myjob
    spec:
      containers:
      - name: hello
        image: 10.29.126.101/rf/busybox
        command: ["echo", "hello k8s job!"]
      restartPolicy: Never
```

![image-20220602114449057](http://myimg.go2flare.xyz/img/image-20220602114449057.png)

1. batch/v1是当前job的Version
2. 指定当前资源的类型时Job
3. restartPolicy是指当前的重启策略。对于 Job，只能设置为 **`Never`** 或者 **`OnFailure`**。对于其他 controller（比如 Deployment）可以设置为 **`Always`** 

启动这个job

```
[root@node00 k8s]# k apply -f myjob.yml -n rf
job.batch/myjob created
```

kubectl get job 查看这个job

```
[root@node00 k8s]# k get job -n rf
NAME    COMPLETIONS   DURATION   AGE
myjob   1/1           6s         20s
```

```
completions为 1/1 表示成功运行了这个job
```

kubectl get pod 查看pod的状态

```
[root@node00 k8s]# k get pod -n rf
NAME          READY   STATUS      RESTARTS   AGE
myjob-dd2zs   0/1     Completed   0          27s
```

看到 状态为Completed表示这个job已经运行完成

kubectl logs 命令查看这个 pod的日志

```
[root@node00 k8s]# k logs myjob-dd2zs -n rf
hello k8s job!
```



## Job的执行失败

```
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob-fail
spec: 
  template: 
    metadata: 
      name: myjob-fail
    spec:
      containers:
      - name: hello
        image: 10.29.126.101/rf/busybox
        command: ["invalid_command", "hello k8s job!"]
      restartPolicy: OnFailure
```

##  ![image-20220602114527241](http://myimg.go2flare.xyz/img/image-20220602114527241.png)

将配置文件中的正确命令 echo 换成一个不存在的错误命令 invalid_command

```
[root@node00 k8s]# k get job -n rf
NAME         COMPLETIONS   DURATION   AGE
myjob-fail   0/1           13s        13s
[root@node00 k8s]# k get pod -n rf
NAME               READY   STATUS              RESTARTS   AGE
myjob-fail-zpct5   0/1     RunContainerError   2          51s
```

 查看job的时候发现COMPLETIONS 0/1 这个job没有完成。但是我们的重启策略是never，所以他不会重启，如果我们将策略设置为 onFaillure，重新运行job

```
[root@node00 k8s]# k delete job myjob-fail -n rf
job.batch "myjob-fail" deleted
[root@node00 k8s]# k apply -f myjob-fail.yml -n rf
job.batch/myjob-fail created
```

通过kubectl get pod 查看pods

发现81s 容器重启3次

```
[root@node00 k8s]# k get pod -n rf
NAME               READY   STATUS             RESTARTS   AGE
myjob-fail-4mtlg   0/1     CrashLoopBackOff   2          65s
[root@node00 k8s]# k get pod -n rf
NAME               READY   STATUS             RESTARTS   AGE
myjob-fail-4mtlg   0/1     CrashLoopBackOff   3          81s
```

通过 kubectl describe pod 查看pod的日志

![image-20220602115231637](http://myimg.go2flare.xyz/img/image-20220602115231637.png)

如果容器启动失败，它会自动回退并重启。

先前的版本是一个容器重启失败，他会继续开启一个新的容器，他检测completions 是否为1 如果不为1，就会一直开启新的容器，这样做非常浪费资源，而且有可能会让CPU内存等资源耗尽，新版本做了改进不会一直创建，而是一直回退重启这个失败的容器。并且重启次数达到一定次数，会自动删除这个容器不在执行这个job。我认为这是一个很聪明的改进。

```
[root@node00 k8s]# k get job -n rf
NAME         COMPLETIONS   DURATION   AGE
myjob-fail   0/1           8m9s       8m9s
[root@node00 k8s]# k get pod -n rf
No resources found in rf namespace.
```



## 并行执行job

有时，我们希望能同时运行多个 Pod，提高 Job 的执行效率。这个可以通过 `parallelism`设置。

```
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob-paral
spec: 
  # 并行job
  parallelism: 2
  # 执行次数
  completions: 5
  template: 
    metadata: 
      name: myjob-paral
    spec:
      containers:
      - name: hello
        image: 10.29.126.101/rf/busybox
        command: ["echo", "hello k8s job!"]
      restartPolicy: OnFailure
```

![image-20220602120251205](http://myimg.go2flare.xyz/img/image-20220602120251205.png)

重新启动 Job 查看到 completions 变为了5

通过查看pod 创建的时间 发现 总有个两个相近完成时间创建的pod ，验证了并发执行为2的设置

上面的例子只是为了演示 Job 的并行特性，实际用途不大。不过现实中确实存在很多需要并行处理的场景。比如批处理程序，每个副本（Pod）都会从任务池中读取任务并执行，副本越多，执行时间就越短，效率就越高。这种类似的场景都可以用 Job 来实现。



## 定时执行Job

Linux 中有 cron 程序定时执行任务，Kubernetes 的 CronJob 提供了类似的功能，可以定时执行 Job。

CronJob 配置文件cronjob.yml示例如下：

```
apiVersion: batch/v2alpha1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["echo","hello k8s job!"]
          restartPolicy: OnFailure
```

![image-20220602121648438](http://myimg.go2flare.xyz/img/image-20220602121648438.png)

1.  `batch/v2alpha1` 是当前 CronJob 的 `apiVersion`。
2.  指明当前资源的类型为 `CronJob`。
3. `schedule` 指定什么时候运行 Job，其格式与 Linux cron 一致。这里 `*/1 * * * *` 的含义是每一分钟启动一次。
4.  `jobTemplate` 定义 Job 的模板，格式与前面 Job 一致。

 执行后报如下错误：

```
[root@k8s-master k8s]# kubectl apply -f cronjob.yml 
error: unable to recognize "cronjob.yml": no matches for kind "CronJob" in version "batch/v2alpha1"
```

我们只需要更改一下/etc/kubernetes/manifests/kube-apiserver.yaml，增加如下配置，重启 kubelet服务，然后重新执行这个cronJob

![img](https://img2018.cnblogs.com/blog/1215197/201811/1215197-20181104103317434-1722957076.png)

```
[root@k8s-master k8s]# systemctl restart kubelet
```

确认这个api已被加载 

![img](https://img2018.cnblogs.com/blog/1215197/201811/1215197-20181104104019042-2106997516.png)

再次执行这个cronjob

```
[root@k8s-master k8s]# kubectl apply -f cronjob.yml 
cronjob.batch/hello created
```

通过`kubectl get cronjob`查看cornjob

```
[root@node00 job]# k get cronjob -n rf
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     1        6s              35s
```

等待几分钟，然后通过 `kubectl get jobs` 查看 Job 的执行情况

通过AGE发现 每个job都比之前的多了 60s，正好符合我们的预期：

![image-20220602121922261](http://myimg.go2flare.xyz/img/image-20220602121922261.png)

**删除cronjob**定时任务

```shell
kubectl delete cronjob hello
```

## 小结

运行容器化应用是 Kubernetes 最重要的核心功能。为满足不同的业务需要，Kubernetes 提供了多种 Controller，包括 **Deployment**、**DaemonSet**、**Job**、**CronJob** 等

# 存储pv/pvc

PV是对底层网络共享存储的抽象，将共享存储定义为一种“资源”，比如Node也是容器应用可以消费的资源。PV由管理员创建和配置，与共享存储的具体实现直接相关。

PVC则是用户对存储资源的一个“申请”，就像Pod消费Node资源一样，PVC能够消费PV资源。PVC可以申请特定的存储空间和访问模式。

StorageClass，用于标记存储资源的特性和性能，管理员可以将存储资源定义为某种类别，正如存储设备对于自身的配置描述（Profile）。根据StorageClass的描述可以直观的得知各种存储资源的特性，就可以根据应用对存储资源的需求去申请存储资源了。存储卷可以按需创建。

## **PV**

PV作为存储资源，主要包括存储能力、访问模式、存储类型、回收策略、后端存储类型等关键信息的设置。

```javascript
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:  #容量
    storage: 5Gi
  accessModes:  #访问模式
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle  #回收策略
  storageClassName: slow  
  nfs:
    path: /
    server: 172.17.0.2
```

复制

kubernetes支持的PV类型如下：

◎ AWSElasticBlockStore：AWS公有云提供的ElasticBlockStore。

◎ AzureFile：Azure公有云提供的File。

◎ AzureDisk：Azure公有云提供的Disk。

◎ CephFS：一种开源共享存储系统。

◎ FC（Fibre Channel）：光纤存储设备。

◎ FlexVolume：一种插件式的存储机制。

◎ Flocker：一种开源共享存储系统。

◎ GCEPersistentDisk：GCE公有云提供的PersistentDisk。

◎ Glusterfs：一种开源共享存储系统。

◎ HostPath：宿主机目录，仅用于单机测试。

◎ iSCSI：iSCSI存储设备。

◎ Local：本地存储设备，目前可以通过指定块（Block）设备提供Local PV，或通过社区开发的sig-storage-local-static-provisioner插件（ https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner ）来管理Local PV的生命周期。

◎ NFS：网络文件系统。

◎ Portworx Volumes：Portworx提供的存储服务。

◎ Quobyte Volumes：Quobyte提供的存储服务。

◎ RBD（Ceph Block Device）：Ceph[块存储](https://cloud.tencent.com/product/cbs?from=10680)。

◎ ScaleIO Volumes：DellEMC的存储设备。

◎ StorageOS：StorageOS提供的存储服务。

◎ VsphereVolume：VMWare提供的存储系统。

### **关键配置参数**

#### **1、存储能力（Capacity）**

描述存储设备具备的能力，支持对存储空间的设置（storage=xx）

#### **2、存储卷模式（Volume Mode）**

volumeMode=xx，可选项包括Filesystem（文件系统）和Block（块设备），默认值是FileSystem。

目前有以下PV类型支持块设备类型：

AWSElasticBlockStore、AzureDisk、FC、GCEPersistentDisk、iSCSI、Local volume、RBD（Ceph Block Device）、VsphereVolume（alpha）

```javascript
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Block
  fc:
    targetWWNs: ["url"]
    lun: 0
    readOnly: false
```

复制

#### **3、访问模式（Access Modes）**

用于描述应用对存储资源的访问权限。

◎ ReadWriteOnce（**RWO**）：读写权限，并且只能被单个Node挂载。

◎ ReadOnlyMany（**ROX**）：只读权限，允许被多个Node挂载。

◎ ReadWriteMany（**RWX**）：读写权限，允许被多个Node挂载。

#### **4、存储类别（Class）**

设定存储的类别，通过storageClassName参数指定给一个StorageClass资源对象的名称，具有特定类别的PV只能与请求了该类别的PVC进行绑定。未绑定类别的PV则只能与不请求任何类别的PVC进行绑定。

#### **5、回收策略（Reclaim Policy）**

通过persistentVolumeReclaimPolicy字段设置，

◎ Retain 保留：保留数据，需要手工处理。

◎ Recycle 回收空间：简单清除文件的操作（例如执行rm -rf /thevolume/* 命令）。

◎ Delete 删除：与PV相连的后端存储完成Volume的删除操作

EBS、GCE PD、Azure Disk、OpenStack Cinder等设备的内部Volume清理）。

#### **6、挂载参数（Mount Options）**

在将PV挂载到一个Node上时，根据后端存储的特点，可能需要设置额外的挂载参数，可根据PV定义中的mountOptions字段进行设置。

```javascript
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  mountOptions:
  - hard
  - nolock
  - nfsvers=3
  gcePersistentDisk:
    fsType: ext4
    pdName: gce-disk-1
```

复制

目前，以下PV类型支持设置挂载参数：

AWSElasticBlockStore、AzureDisk、AzureFile、CephFS、Cinder (OpenStack block storage)、GCEPersistentDisk、Glusterfs、NFS、Quobyte Volumes、RBD (Ceph Block Device)、StorageOS、VsphereVolume、iSCSI

#### **7、节点亲和性（Node Affinity）**

限制只能通过某些Node来访问Volume，可在nodeAffinity字段中设置。使用这些Volume的Pod将被调度到满足条件的Node上。

此参数仅用于Local存储卷上。

```javascript
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - my-node
```

复制

### **PV生命周期的各个阶段**

◎ Available：可用状态，还未与某个PVC绑定。

◎ Bound：已与某个PVC绑定。

◎ Released：绑定的PVC已经删除，资源已释放，但没有被集群回收。

◎ Failed：自动资源回收失败。

## **PVC**

PVC作为用户对存储资源的需求申请,主要包括存储空间请求、访问模式、PV选择条件和存储类别等信息的设置。

```javascript
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes:  #访问模式
  - ReadWriteOnce
  resources: #申请资源，8Gi存储空间
    requests:
      storage: 8Gi
  storageClassName: slow #存储类别
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
    - {key: environment, operator: In, values: [dev]}
```

复制

### **关键配置**

#### **资源请求（Resources）**

描述对存储资源的请求，目前仅支持request.storage的设置，即是存储空间的大小

#### **访问模式（AccessModes）**

用于描述对存储资源的访问权限，与PV设置相同

#### **存储卷模式（Volume Modes）**

用于描述希望使用的PV存储卷模式，包括文件系统和块设备。

#### **PV选择条件（Selector）**

通过对Label Selector的设置，可使PVC对于系统中已存在的各种PV进行筛选。

选择条件可以使用matchLabels和matchExpressions进行设置，如果两个字段都设置了，则Selector的逻辑将是两组条件同时满足才能完成匹配

#### **存储类别（Class）**

PVC 在定义时可以设定需要的后端存储的类别（通过storageClassName字段指定），以减少对后端存储特性的详细信息的依赖。只有设置了该Class的PV才能被系统选出，并与该PVC进行绑定

PVC也可以不设置Class需求。如果storageClassName字段的值被设置为空（storageClassName=""），则表示该PVC不要求特定的Class，系统将只选择未设定Class的PV与之匹配和绑定。PVC也可以完全不设置storageClassName字段，此时将根据系统是否启用了名为DefaultStorageClass的admission controller进行相应的操作

#### **未启用DefaultStorageClass**

等效于PVC设置storageClassName的值为空（storageClassName=""），即只能选择未设定Class的PV与之匹配和绑定。

#### **启用DefaultStorageClass**

要求集群管理员已定义默认的StorageClass。如果在系统中不存在默认StorageClass，则等效于不启用DefaultStorageClass的情况。如果存在默认的StorageClass，则系统将自动为PVC创建一个PV（使用默认StorageClass的后端存储），并将它们进行绑定。集群管理员设置默认StorageClass的方法为，在StorageClass的定义中加上一个annotation“storageclass.kubernetes.io/is-default-class= true”。如果管理员将多个StorageClass都定义为default，则由于不唯一，系统将无法为PVC创建相应的PV。

PVC和PV都受限于Namespace，PVC在选择PV时受到Namespace的限制，只有相同Namespace中的PV才可能与PVC绑定。Pod在引用PVC时同样受Namespace的限制，只有相同Namespace中的PVC才能挂载到Pod内。

当Selector和Class都进行了设置时，系统将选择两个条件同时满足的PV与之匹配。

另外，如果资源供应使用的是动态模式，即管理员没有预先定义PV，仅通过StorageClass交给系统自动完成PV的动态创建，那么PVC再设定Selector时，系统将无法为其供应任何存储资源。

在启用动态供应模式的情况下，一旦用户删除了PVC，与之绑定的PV也将根据其默认的回收策略“Delete”被删除。如果需要保留PV（用户数据），则在动态绑定成功后，用户需要将系统自动生成PV的回收策略从“Delete”改成“Retain”。

## **PV和PVC的生命周期**

![img](https://ask.qcloudimg.com/http-save/yehe-6475469/xudbbyzzxw.png?imageView2/2/w/1620)

将PV看作可用的存储资源，PVC则是对存储资源的需求。

### **资源供应**

k8s支持两种资源的供应模式：静态模式（Static）和动态模式（Dynamic）。资源供应的结果就是创建好的PV。

静态模式：集群管理员手工创建许多PV，在定义PV时需要将后端存储的特性进行设置。

动态模式：集群管理员无需手工创建PV，而是通过StorageClass的设置对后端存储进行描述，标记为某种类型。此时要求PVC对存储的类型进行声明，系统将自动完成PV的创建及与PVC的绑定。PVC可以声明Class为""，说明该PVC禁止使用动态模式。

### **资源绑定**

在定义好PVC之后，系统将根据PVC对存储资源的要求（存储空间和访问模式）在已存在的PV中选择一个满足PVC要求的PV，一旦找到，就将该PV与定义的PVC进行绑定，应用就可以使用这个PVC了。如果系统中没有这个PV，则PVC则会一直处理Pending状态，直到系统中有符合条件的PV。PV一旦绑定到PVC上，就会被PVC独占，不能再与其他PVC进行绑定。当PVC申请的存储空间比PV的少时，整个PV的空间就都能够为PVC所用，可能会造成资源的浪费。如果资源供应使用的是动态模式，则系统在为PVC找到合适的StorageClass后，将自动创建一个PV并完成与PVC的绑定。

### **资源使用**

Pod使用Volume定义，将PVC挂载到容器内的某个路径进行使用。Volume的类型为Persistent VolumeClaim，在容器挂载了一个PVC后，就能被持续独占使用。多个Pod可以挂载到同一个PVC上。

```javascript
volumes:
  - name: pv
    persistentVolumeClaim:
      claimName: pvc
```

### **资源释放**

当存储资源使用完毕后，可以删除PVC，与该PVC绑定的PV会被标记为“已释放”，但还不能立刻与其他PVC进行绑定。通过之前PVC写入的数据可能还被保留在存储设备上，只有在清除之后该PV才能被再次使用。

### **资源回收**

对于PV，管理员可以设定回收策略，用于设置与之绑定的PVC释放资源之后如何处理遗留数据的问题。只有PV的存储空间完成回收，才能供新的PVC绑定和使用。

通过两张图分别对在静态资源供应模式和动态资源供应模式下，PV、PVC、StorageClass及Pod使用PVC的原理进行说明。

在静态资源供应模式下，通过PV和PVC完成绑定，并供Pod使用的存储管理机制

![img](https://ask.qcloudimg.com/http-save/yehe-6475469/j6iuof3utd.png?imageView2/2/w/1620)

在动态资源供应模式下，通过StorageClass和PVC完成资源动态绑定（系统自动生成PV），并供Pod使用的存储管理机制

![img](https://ask.qcloudimg.com/http-save/yehe-6475469/3o40ig6gws.png?imageView2/2/w/1620)

## **StorageClass**

StorageClass作为对存储资源的抽象定义，对用户设置的PVC申请屏蔽后端存储的细节，一方面减少了用户对存储资源细节的关注，另一方面减少了管理员手工管理PV的工作，由系统自动完成PV的创建和绑定，实现了动态的资源供应。

StorageClass的定义主要包括名称、后端存储的提供者（privisioner）和后端存储的相关参数配置。StorageClass一旦被创建，就无法修改，如需修改，只能删除重建。

下例定义了一个名为standard的StorageClass，提供者为aws-ebs，其参数设置了一个type，值为gp2：

```javascript
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
privisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

复制

### **关键配置**

1、提供者（Privisioner）

描述存储资源的提供者，也可以看作后端存储驱动。

2、参数（Parameters）

后端存储资源提供者的参数设置，不同的Provisioner包括不同的参数设置。某些参数可以不显示设定，Provisioner将使用其默认值。

### **设置默认的StorageClass**

首先需要启用名为DefaultStorageClass的admission controller，即在kube-apiserver的命令行参数--admission-control中增加

```javascript
--admission-control=...,DefaultStorageClass
```

复制

然后，在StorageClass的定义中设置一个annotation：

```javascript
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
  annotations:
    storageclass.beta.kubernetes.io/is-default-class="true"
privisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

通过kubectl create命令创建成功后，查看StorageClass列表，可以看到名为gold的StorageClass被标记为default：

```javascript
kubectl get sc
```

## **CSI存储机制**

Container Storage Interface（CSI）机制，用于在kubenetes和外部存储系统之间建立一套标准的存储管理接口，通过该接口为容器提供存储服务。

### **CSI存储插件的关键组件和部署架构**

![img](https://ask.qcloudimg.com/http-save/yehe-6475469/68a41flv2v.png?imageView2/2/w/1620)

主要包括两个组件：CSI Controller和CSI Node

#### **1、CSI Controller**

提供存储服务视角对存储资源和存储进行管理和操作，在k8s中建议部署为单实例Pod，可以使用StatefulSet和Deployment控制器进行部署，设置副本数为1，保证为一种存储插件只运行一个控制器实例。

在此Pod内部署两个容器：

（1）与Master（kubernetes Controller Manager）通信的辅助sidecar容器，

在sidecar容器内又可以包含extenal-attacher和extenal-privisioner两个容器，功能如下：

◎ external-attacher：监控VolumeAttachment资源对象的变更，触发针对CSI端点的ControllerPublish和ControllerUnpublish操作。

◎ external-provisioner：监控PersistentVolumeClaim资源对象的变更，触发针对CSI端点的CreateVolume和DeleteVolume操作

（2）CSI Driver存储驱动容器，第三方提供，需实现上述接口

这两个容器使用本地Socket（Unix Domain Socket，UDS），并使用gPRC协议进行通信，sidecar容器通过socket调用CSI Driver容器的CSI接口，CSI Driver容器负责具体的存储卷操作。

#### **2、CSI Node**

对主机（Node）上的Volume进行管理和操作，建议使用DaemonSet，每个Node上都运行一个Pod。

在此Pod上部署如下两个容器：

（1）与kubelet通信的辅助sidecar容器node-driver-register，主要功能是将存储驱动注册到kubelet中。

（2）CSI Driver存储驱动容器，由第三方存储提供商提供，主要功能是接收kubelet的调用，需要实现一系列与Node相关的CSI接口，例如NodePublishVolume接口（用于将Volume挂载到容器内的目标路径）、NodeUnpublishVolume接口（用于从容器中卸载Volume），等等。

node-driver-register容器与kubelet通过Node主机的一个hostPath目录下的unix Socket进行通信，CSI Driver容器与kubelet通过Node主机的另一个hostPath目录下的unix Socket进行通信，同时需要将kubelet的工作目录（/var/lib/kubelet）挂载到CSI Driver容器，用于为Pod进行Volume的管理操作（包括mount，unmount等）。



### [k8s pv 的三种挂载模式](https://www.cnblogs.com/37yan/p/7833128.html)

ReadWriteOnce（**RWO**）：可读可写，只能被一个Node节点挂载

ReadWriteMany（**RWX**）：可读可写，可以被多个Node节点挂载

ReadOnlyMany（**ROX**）：只读，能被多个Node节点挂载

### 挂载pv

```
volumeMounts:
- name: otel-collector-config-vol
  mountPath: /conf
- name: otel-collector-filePath # 使用宿主机的文件夹
  mountPath: /usr/pro_repo/obsv_test/data
volumes:
- configMap:
    name: otel-collector-conf
    items:
      - key: otel-collector-config
        path: otel-collector-config.yaml
  name: otel-collector-config-vol
- name: otel-collector-filePath
  hostPath: 
    path: /usr/pro_repo/obsv_test/data
    type: DirectoryOrCreate
```

## [SubPath](https://www.jianshu.com/p/cafb7aee7bc4)

subPath的使用方法一共有两种：
 1. 同一个**pod中多容器**挂载同一个卷的**同一个目录**时**提供多容器间隔离**
 1. 将**configMap**和**secret**作为文件**挂载到容器中**而**不覆盖挂载目录下相互的文件**

本文主要解释第一点，按照k8s官网的解释，subPath在是挂载卷中的存储目录，不指定默认存储在卷的根目录

![img](https://upload-images.jianshu.io/upload_images/24299681-32cb2a59824bf4dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/944/format/webp)

首先创建一个包含两个container的pod，对应的yaml文件如下，两个container挂载目录下的文件都会存储在卷的根目录下

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: podstest-volume
  labels:
    app: podstest
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: daocloud-nfs
  
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: multipod
  labels:
    app: podstest
spec:
  replicas: 1
  selector:
    matchLabels:
      app: podstest
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: podstest
    spec:
      containers:
        - name: test-container1
          image: 10.29.3.30:30004/rf/busybox
          imagePullPolicy: IfNotPresent
          command: [ "/bin/sh", "-c", "sleep 36000"]
          volumeMounts:
            - name: podstest
              mountPath: /etc/volume-file
        - name: test-container2
          image: 10.29.3.30:30004/rf/busybox
          imagePullPolicy: IfNotPresent
          command: [ "/bin/sh", "-c", "sleep 36000"]
          volumeMounts:
            - name: podstest
              mountPath: /etc/volume-file
      volumes:
        - name: podstest
          persistentVolumeClaim:
            claimName: podstest-volume
```

接下来做一个简单的验证，首先进到test-container中在/etc/volume-file下创建一个a.txt的文件，退出之后进入test-container2的/etc/volume-file目录，因为**未指定subPath**，文件会直接**存储在卷的根目录**下，所以在test-container2下**可以看见test-container刚刚新建的a.txt文件**，两容器间的所使用的**挂载目录是相通的**。

```shell
[root@node00 pv]# k exec -it multipod-85f8df867b-9phml -c test-container1 -n rf -- sh
/ # cd /etc/volume-file
/etc/volume-file # cat >a.txt <<end
> hey, nihao
> end
/etc/volume-file # exit 
[root@node00 pv]# k exec -it multipod-85f8df867b-9phml -c test-container2 -n rf -- sh
/ # cd /etc/volume-file
/etc/volume-file # ls
a.txt
/etc/volume-file # cat a.txt
hey, nihao
```

![image-20220602170221125](http://myimg.go2flare.xyz/img/image-20220602170221125.png)

接下来使用subpath做两个容器的挂载目录的隔离，将部署的yaml以下位置中添加subpath

```shell
[root@node00 pv]# k exec -it multipod-sp-5d4b899dc9-bm65t -c test-container1 -n rf -- sh
/ #  cd /etc/volume-file
/etc/volume-file # cat >a.txt
hi,nihao
^C
/etc/volume-file # ls
a.txt
/etc/volume-file # exit
[root@node00 pv]# k exec -it multipod-sp-5d4b899dc9-bm65t -c test-container2 -n rf -- sh
/ # cd /etc/volume-file
/etc/volume-file # ls
/etc/volume-file # cat >b.txt <<end
> hey, nihao
> end
/etc/volume-file # ls
b.txt
```

![image-20220602172417134](http://myimg.go2flare.xyz/img/image-20220602172417134.png)

subpath已经起到了**两个容器间的隔离作用**，查看集群中实际被挂载的文件目录，可以看到隔离两个容器的subpath目录

![image-20220602173042426](http://myimg.go2flare.xyz/img/image-20220602173042426.png)



## 补充说明

### daocloud集群推荐使用

![image-20220601153732350](http://myimg.go2flare.xyz/img/image-20220601153732350.png)

直接使用已经声明好的storageClassName：daocloud-nfs，作为PVC的存储卷，会自动在nfs上分配需要的储存块供PVC使用，所以只用声明PVC中指定储存类，既可以自动分配储存类绑定的nfs，在nfs地址的文件服务器中可以看到自动生成的文件目录

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-claim
  labels:
    app: gitlab
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: daocloud-nfs
```

![image-20220601185219720](http://myimg.go2flare.xyz/img/image-20220601185219720.png)



## endpoint

　　endpoint是k8s集群中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址。service配置selector，endpoint controller才会自动创建对应的endpoint对象；否则，不会生成endpoint对象.

例如，k8s集群中创建一个名为hello的service，就会生成一个同名的endpoint对象，ENDPOINTS就是service关联的pod的ip地址和端口。

　　一个 Service 由一组 backend Pod 组成。这些 Pod 通过 `endpoints` 暴露出来。 Service Selector 将持续评估，结果被 POST 到一个名称为 Service-hello 的 Endpoint 对象上。 当 Pod 终止后，它会自动从 Endpoint 中移除，新的能够匹配上 Service Selector 的 Pod 将自动地被添加到 Endpoint 中。 检查该 Endpoint，注意到 IP 地址与创建的 Pod 是相同的。现在，能够从集群中任意节点上使用 curl 命令请求 hello Service `<CLUSTER-IP>:<PORT>` 。 注意 Service IP 完全是虚拟的，它从来没有走过网络，如果对它如何工作的原理感到好奇，可以阅读更多关于 [服务代理](https://kubernetes.io/docs/user-guide/services/#virtual-ips-and-service-proxies) 的内容。

　　Endpoints是实现实际服务的端点集合。

Kubernetes在创建Service时，根据Service的标签选择器（Label Selector）来查找Pod，据此创建与Service同名的EndPoints对象。当Pod的地址发生变化时，EndPoints也随之变化。Service接收到请求时，就能通过EndPoints找到请求转发的目标地址。

Service不仅可以代理Pod，还可以代理任意其他后端，比如运行在Kubernetes外部Mysql、Oracle等。这是通过定义两个同名的service和endPoints来实现的。

在实际的生产环境使用中，通过分布式存储来实现的磁盘在mysql这种IO密集性应用中，性能问题会显得非常突出。所以在实际应用中，一般不会把mysql这种应用直接放入kubernetes中管理，而是使用专用的服务器来独立部署。而像web这种无状态应用依然会运行在kubernetes当中，这个时候web服务器要连接kubernetes管理之外的数据库，有两种方式：一是直接连接数据库所在物理服务器IP，另一种方式就是借助kubernetes的Endpoints直接将外部服务器映射为kubernetes内部的一个服务。

　　简单认为：动态存储pod名字与pod ip对应关系的list，并提供将请求转发到实际pod上的能力

## service

```
---
apiVersion: v1
kind: Service
metadata:
  name: kube-node-service
  labels:
    name: kube-node-service
spec:
  type: NodePort      #这里代表是NodePort类型的
  ports:
  - port: 8080        #这里的端口和clusterIP对应，即ip:8080,供内部访问。
    targetPort: 8080  #端口一定要和container暴露出来的端口对应
    protocol: TCP
    nodePort: 32143   # 所有的节点都会开放此端口，此端口供外部调用。
  selector:
    app: web          #这里选择器一定要选择容器的标签，之前写name:kube-node是错的。
```

```
集群configMap，deployment，service，secret等导出和导入的方法，以service为例，其他同理：

单个service：

     导出: kubectl get service serviceName -o yaml > backup.yaml

     导入: kubectl create -f backup.yaml

所有service：

     导出: kubectl get service -o yaml > backup.yaml

     导入: kubectl create -f backup.yaml
```

# 部署实例

## gitlab(ce/ee)

### pv/pvc

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-claim
  labels:
    app: gitlab
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: daocloud-nfs
```

### redis

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    app: gitlab
spec:
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: gitlab
    tier: backend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: gitlab
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab
      tier: backend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: gitlab
        tier: backend
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: gitlab/middleware
                    operator: Exists
      tolerations:
        - key: gitlab/middleware
          effect: NoSchedule
      containers:
        - image: 10.29.3.30:30004/gitlab/redis:5.0.7
          name: redis
          resources:
            requests:
              cpu: 1
              memory: 1Gi
            limits:
              cpu: 1
              memory: 1Gi
          ports:
            - containerPort: 6379
              name: redis
          volumeMounts:
            - name: redis
              mountPath: /data
              subPath: redis
      volumes:
        - name: redis
          persistentVolumeClaim:
            claimName: gitlab-claim
```

### postgres

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgresql
  labels:
    app: gitlab
spec:
  ports:
    - port: 5432
  selector:
    app: gitlab
    tier: postgreSQL
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgresql
  labels:
    app: gitlab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab
      tier: postgreSQL
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: gitlab
        tier: postgreSQL
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: gitlab/middleware
                    operator: Exists
      tolerations:
        - key: gitlab/middleware
          effect: NoSchedule
      containers:
        - image: 10.29.3.30:30004/gitlab/postgres:12.3
          name: postgresql
          resources:
            requests:
              cpu: 1
              memory: 2Gi
            limits:
              cpu: 1
              memory: 2Gi
          env:
            - name: POSTGRES_USER
              value: gitlab
            - name: POSTGRES_DB
              value: gitlabhq_production
            - name: POSTGRES_PASSWORD
              value: gitlab
          ports:
            - containerPort: 5432
              name: postgresql
          volumeMounts:
            - name: postgresql
              mountPath: /var/lib/postgresql/data
              subPath: postgre
      volumes:
        - name: postgresql
          persistentVolumeClaim:
            claimName: gitlab-claim
```

### gitlab-ce

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gitlab-cm
  
data:
  gitlab: |
    nginx['enable'] = false
    gitlab_rails['trusted_proxies'] = ['10.29.3.30']
    gitlab_workhorse['listen_network'] = "tcp"
    gitlab_workhorse['listen_addr'] = "0.0.0.0:38000"
    prometheus['enable'] = false
    alertmanager['enable'] = false
    grafana['enable'] = false
    postgresql['enable'] = false
    gitlab_rails['db_username'] = "gitlab"
    gitlab_rails['db_password'] = "gitlab"
    gitlab_rails['db_host'] = "postgresql"
    gitlab_rails['db_port'] = "5432"
    gitlab_rails['db_database'] = "gitlabhq_production"
    gitlab_rails['db_adapter'] = 'postgresql'
    gitlab_rails['db_encoding'] = 'utf8'
    redis['enable'] = false
    gitlab_rails['redis_host'] = 'redis'
    gitlab_rails['redis_port'] = '6379'
    gitlab_rails['gitlab_shell_ssh_port'] = 38022
    external_url 'http://10.29.3.30:38000'
    user['uid'] = 9000
    user['gid'] = 9000
    web_server['uid'] = 9001
    web_server['gid'] = 9001
    registry['uid'] = 9002
    registry['gid'] = 9002
    gitlab_rails['omniauth_enabled'] = true
    gitlab_rails['omniauth_allow_single_sign_on'] = ['saml']
    # gitlab_rails['omniauth_auto_sign_in_with_provider'] = 'saml'
    gitlab_rails['omniauth_block_auto_created_users'] = false
    gitlab_rails['omniauth_auto_link_saml_user'] = true
    gitlab_rails['omniauth_providers'] = [
        {
            name: 'saml',
            args: {
                assertion_consumer_service_url: 'http://10.29.3.30:38000/users/auth/saml/callback',
                idp_cert: " -----BEGIN CERTIFICATE-----
    \nMIICnTCCAYUCBgGAzOEFWTANBgkqhkiG9w0BAQsFADASMRAwDgYDVQQDDAdkeC1hcmNoMB4XDTIyMDUxNjEyMzcyMFoXDTMyMDUxNjEyMzkwMFowEjEQMA4GA1UEAwwHZHgtYXJjaDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKfNmexdb91H/0D/p4EiPvdVp6H2hEuJ+32o1cFRZGTiLtuXWLWAT4Dtg5MuVhnrx2/FBFy8bMeIeue+YGVRHCDnKGWY/cZ4LEq0QCV62uaS+KlbycW1PzYzdVfdwWeDtAoHT9rkqVAzxXcBi2VYEKfMdOy/Js/TS7Bmarcrz6Fz4Nkb1Se89F4fyEf7XPeiqpbU7nFfPqhQg2pxwzV7eZo8VUkDWRANRxpPnpubaszfSUz7iBx5pmFWQDpNIOHw3LVE8MXrZqS8SHd/uOQNPVTq9q+Nk5F5khtUsjHGb/v00bZroZvn4kRLRQTzayPNXL5ZCk13CWQE2lPPZjnPXQMCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAKK9B8M5z2LqDxV3paCvWU5mk99yBs8ybOON4Ce8bk0lljLDWt9bAlC4IZCTUJdMp1KCpplswyNjOyUVzF5if33kZCA3Cr2inv96JQmwZpyXvIRjqTq0KZlPkPPY2dXhNeLhsi8AtfRH9tWPMjuCUNMpVczCnmxqGUKdV6XTQXWqIWg2EzGWjJL/dfyIgAXYKawbfTyIsRGND0KPSMbIGDdiaqvddQ3JL+IfNcvovPvFrltYiLKBC/7bNGBdXI5l1iOH3b72XvbF1zs0Z/1kTXDqabLcrKocUIpKlzl1YZChRRQn4GTshVgpeVkkmRbHX6y2VXoNIXyZc9/fJkulh7g==\n -----END CERTIFICATE----- \n",
                     idp_sso_target_url: 'http://10.29.3.30:30035/auth/realms/dx-arch/protocol/saml/clients/gitlab',
                issuer: 'gitlab',
                name_identifier_format: 'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent'
            },
            label: 'DX-ARCH'
        }
    ]

---
apiVersion: v1
kind: Service
metadata:
  name: gitlab
  labels:
    app: gitlab
spec:
  type: NodePort
  ports:
    - name: gitlab-ui
      port: 38000
      protocol: TCP
      targetPort: 38000
      nodePort: 38000
    - name: gitlab-ssh
      port: 22
      protocol: TCP
      targetPort: 22
      nodePort: 38022
  selector:
    app: gitlab
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab
  labels:
    app: gitlab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab
      tier: frontend
  template:
    metadata:
      name: gitlab
      labels:
        app: gitlab
        tier: frontend
    spec:
      tolerations:
        - key: gitlab/application
          effect: NoSchedule
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: gitlab/application
                    operator: Exists
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - gitlab
                  - key: tier
                    operator: In
                    values:
                      - frontend
              namespaces:
                - gitlab
              topologyKey: kubernetes.io/hostname
      containers:
        - image: 10.29.3.30:30004/gitlab/gitlab-ce:latest
          name: gitlab
          resources:
            limits:
              cpu: 1
              memory: 4Gi
            requests:
              cpu: 1
              memory: 4Gi
          env:
            - name: GITLAB_OMNIBUS_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: gitlab-cm
                  key: gitlab
          ports:
            - containerPort: 38000
              name: gitlab-ui
            - containerPort: 22
              name: gitlab-ssh
          volumeMounts:
            - name: gitlab
              mountPath: /var/opt/gitlab/git-data
              subPath: gitlab-data/git-data
            - name: gitlab
              mountPath: /etc/gitlab
              subPath: gitlab-configuration
            - name: gitlab
              mountPath: /var/opt/gitlab/gitlab-rails/uploads
              subPath: gitlab-data/gitlab-rails/uploads
            - name: gitlab
              mountPath: /var/opt/gitlab/gitlab-rails/shared
              subPath: gitlab-data/gitlab-rails/shared
            - name: gitlab
              mountPath: /var/opt/gitlab/gitlab-ci/builds
              subPath: gitlab-data/gitlab-ci/builds
            - name: gitlab
              mountPath: /var/opt/gitlab/.ssh
              subPath : gitlab-data/.ssh
      volumes:
        - name: gitlab
          persistentVolumeClaim:
            claimName: gitlab-claim
```

### job

```yaml
---
apiVersion: batch/v1
kind: Job
metadata:
  name: gitlab-init
spec:
  template:
    metadata:
      name: gitlab-init
    spec:
      restartPolicy: Never
      tolerations:
        - key: gitlab/application
          effect: NoSchedule
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: gitlab/application
                    operator: Exists
      containers:
        - image: 10.29.3.30:30004/gitlab/gitlab-ce:latest
          command: ["/bin/bash"]
          args: ["-c", "/assets/wrapper; touch /etc/gitlab/skip-auto-reconfigure"]
          name: install
          resources:
            limits:
              cpu: 1
              memory: 3Gi
            requests:
              cpu: 1
              memory: 3Gi
          env:
            - name: GITLAB_OMNIBUS_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: gitlab-cm
                  key: gitlab
          volumeMounts:
            - name: gitlab
              mountPath: /var/opt/gitlab/git-data
              subPath: gitlab-data/git-data
            - name: gitlab
              mountPath: /etc/gitlab
              subPath: gitlab-configuration
            - name: gitlab
              mountPath: /var/opt/gitlab/gitlab-rails/uploads
              subPath: gitlab-data/gitlab-rails/uploads
            - name: gitlab
              mountPath: /var/opt/gitlab/gitlab-rails/shared
              subPath: gitlab-data/gitlab-rails/shared
            - name: gitlab
              mountPath: /var/opt/gitlab/gitlab-ci/builds
              subPath: gitlab-data/gitlab-ci/builds
            - name: gitlab
              mountPath: /var/opt/gitlab/.ssh
              subPath : gitlab-data/.ssh
      volumes:
        - name: gitlab
          persistentVolumeClaim:
            claimName: gitlab-claim
```



## eureka

```yaml
apiVersion: v1
kind: Service
metadata:
  name: register-server
  labels:
    service: register-server
spec:
  type: NodePort
  ports:
    - port: 8000
      targetPort: http
      protocol: TCP
      name: http
      nodePort: 38001
  selector:
    service: register-server

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: eureka-data
  labels:
    app: eureka-data
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 4Gi
  storageClassName: daocloud-nfs


---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: register-server
  labels:
    service: register-server
spec:
  replicas: 3
  serviceName: register-server
  selector:
    matchLabels:
      service: register-server
  template:
    metadata:
      labels:
        service: register-server
      annotations:
        service: register-server
    spec:
      containers:
        - name: register-server
          image: 10.29.3.30:30004/api/eureka:0.1.0
          imagePullPolicy: Always
          env:
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          ports:
            - name: http
              containerPort: 8000
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8001
              scheme: HTTP
            failureThreshold: 3
            initialDelaySeconds: 60
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          resources:
            limits:
              cpu: 1
              memory: 1.7Gi
            requests:
              cpu: 1
              memory: 1.2Gi
          volumeMounts:
          - mountPath: /Charts
            name: eureka-data
      volumes:
      - name: eureka-data
  podManagementPolicy: "Parallel"
```



# 常见问题

## connect refused

检查service的selector的label和deploy的label是否一致

```
[root@dce-10-23-12-108 prom_test]# curl 172.31.79.236:32323
curl: (7) Failed to connect to 172.31.79.236 port 32323: Connection refused
```

```
Name:              prom-test-exporter
Namespace:         rftest
Labels:            <none>
Annotations:       Selector:  app=prom-test-exporter
Type:              ClusterIP
IP:                172.31.79.236
IPFamily:          IPv4
Port:              prom-test-exporter  32323/TCP
TargetPort:        32323/TCP
Endpoints:         172.31.79.236:32323
Session Affinity:  None
Events:            <none>
```

```
selector:
    matchLabels:
      k8s-app: prom-test-exporter
selector中的matchLabels一定要和service的一致，否则会报Connection refused错误
```

## 节点长时间pending

```
Events:
  Type     Reason                  Age               From               Message
  ----     ------                  ----              ----               -------
  Warning  FailedScheduling        39s               default-scheduler  0/7 nodes are available: 2 Insufficient memory, 3 Insufficient cpu, 4 node(s) didn't match node selector.
  Warning  FailedScheduling        39s               default-scheduler  0/7 nodes are available: 2 Insufficient memory, 3 Insufficient cpu, 4 node(s) didn't match node selector.

```

```
kubectl  get node --show-labels=true
```

```
k #打标签
```

3.把pod调度到指定标签
启动一个deployment副本数为2,让pod调度到node1

```
[root@apiserver k8s]# cat  selec.yml
1
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-select-node1
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        node: node1
      containers:

   - name: nginx-select-node1
     image: nginx:latest
     ports:
     - containerPort: 80
       ]]
```



发现两个pod都调度到了node1

```
Failed to create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container network for pod : networkPlugin cni failed to set up pod network: failed to request 1 IPv4 addresses. IPAM allocated only 0
```



## k8s pod OOMKilled 错误原因

k8s oomkilled 错误原因：容器使用的内存资源超过了限制。只要节点有足够的内存资源，那容器就可以使用超过其申请的内存，但是不允许容器使用超过其限制的资源。
在yaml文件的resources.limits.memory 下定义了容器使用的内存限制，如果容器中的进程使用内存超过这个限制，就会出现oomkilled错误，容器被终止。

```
resources:
  limits:
    cpu: '8'
    memory: 32Gi
```

解决办法：1.优化程序，使之使用内存大小在范围内  2.修改yaml，增大内存的限制大小

## IPV4无法分配

原因：默认的集群内的ip被分配完，需要自定义ip池，在使用annotation绑定

```
network: failed to request 1 IPv4 addresses. IPAM allocated only 0
```

![image-20220601183926592](http://myimg.go2flare.xyz/img/image-20220601183926592.png)

新建ip池，并使用annotation绑定

![image-20220601184133304](http://myimg.go2flare.xyz/img/image-20220601184133304.png)

annotation

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab
  labels:
    app: gitlab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab
      tier: frontend
  template:
    metadata:
      name: gitlab
      labels:
        app: gitlab
        tier: frontend
      annotations:
        dce.daocloud.io/parcel.egress.burst: '0'
        dce.daocloud.io/parcel.egress.rate: '0'
        dce.daocloud.io/parcel.ingress.burst: '0'
        dce.daocloud.io/parcel.ingress.rate: '0'
        dce.daocloud.io/parcel.net.type: calico
        dce.daocloud.io/parcel.net.value: 'gitlab-ipv4-gitlab2,gitlab-ipv6-gitlab2'
```

