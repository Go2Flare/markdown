# 前言

Keycloak 是一个优秀的**开源身份与访问管理**系统，旨在为现代的应用程序和服务提供包含**身份管理和访问管理**功能。不少企业包含红帽公司，都将其作为其站点的单点登录工具，通过使用 Keycloak，只需要**少量编码甚至不用编码**，就能很容易地使应用程序和服务更**安全**。

而在中国，企业往往通过**微信公众号**和粉丝进行互动，以及通过公众号进行微信营销等等，微信公众号粉丝和企业网站的用户其实有多点登录的场景，因此如果能够将**微信登录接入到单点登录**环节，不仅会方便用户，也会方便企业。但是微信官方的 **OAuth 登录**却是和**公众号独立**的，一个是开放平台，一个是公众平台，虽然可以通过相互绑定之后使用 unionid 关联两个平台里的同一个用户，但是在微信的 OAuth 登录环节，不需要公众号的参与，企业失去了一次和用户互动的机会。

更好的做法是，用户选择**微信登录**，扫码后手机号被**导流到企业公众号**，一旦**新用户关注公众号或者老用户进入到公众号**页面，便能够接收到一条**定制化的欢迎**信息，同时网页站点**自动登录**成功。

## 好处

1、用户体验更好：对于老用户**扫码后即登录**，不需要**二次点击**；对于新用户，只需要点击关注。无论新老用户都能收到可爱的欢迎信息，感觉**亲切**。

2、开发更便捷：不用**对接开放平台**、不用考虑 **unionid**。

3、运营更省心：不用**额外每年认证一次**开放平台，**微信粉丝和网站用户**一一对应。

## 实现效果预览

注意，由于测试公众号的限制，只能支持 100 名用户。如果由于用户超限导致预览不成功，可以放心跳过预览往下看实现细节。

点击这个链接（手机端微信内网页打开或者从桌面网页打开都可以工作）：

[https://keycloak.jiwai.win/auth/realms/UniHeart/login-actions/authenticate?execution=2668dd1a-735d-4fd4-b4ea-400309a4289c&client_id=account&tab_id=oXaMd2IidLE](https://link.zhihu.com/?target=https%3A//keycloak.jiwai.win/auth/realms/UniHeart/login-actions/authenticate%3Fexecution%3D2668dd1a-735d-4fd4-b4ea-400309a4289c%26client_id%3Daccount%26tab_id%3DoXaMd2IidLE)

打开登录界面如下：

![img](https://pic4.zhimg.com/80/v2-b03170db47e3bda035a549274cde411f_1440w.jpg)

可以看到在默认的 Keycloak 登录框右边有一个“微信”选项，点击它可以打开一个二维码，扫码后，会进入到我的测试公众号界面，如果你已经关注，那么会收到一条欢迎信息并成功登录系统；如果你没有关注过，则需要点击关注，然后在收到欢迎信息的同时，成功登录系统。

![img](https://pic4.zhimg.com/80/v2-c72225a6cd9f1060dc6d56bbbb04a123_1440w.jpg)

![img](https://pic3.zhimg.com/80/v2-a2aa3dc6a1b606be1fcd615ec541dcca_1440w.jpg)

以上是成功登录系统后进入到个人账号页面的截图，注意，如果是第一次登录，那么需要验证电子邮件后才能进入这个页面（是否验证电子邮件可以在 Keycloak 中配置）。

# 概念简介

[keycloak](https://www.keycloak.org/) 是一个针对Web应用和RESTfull Web API 提供SSO（Single Sign On：单点登陆），它是一个开源软件，源码地址是：https://github.com/keycloak/keycloak/

核心概念：

users：用户是一个可以登陆系统的实体，它可以拥有联系它们自身的属性，例如邮箱、用户名、地址、电话号码或生日等，可以为user分配组别或者角色。

authentication：相当于密码，可以验证和识别一个user。

authorization：给予用户访问的过程。

credentials：证书，可以供keycloak验证用户的东西，例如密码、一次性密码、证书、指纹等。

roles：相当于用户的一个分类 ，一个组织可能有Admin\user\manager\employee等角色，应用程序经常会分配权限给角色，而不是用户，因为用户太难管理。

user role mapping：定义了一个用户及角色的关系，一个用户可以属于零个或多个角色，用户与角色的映射关系，这样就可以决定用在各种资源访问的权限管理。

composite roles：复合角色可以包含其他的角色，用户拥有了复合角色就相当于拥有了它下面的所有子角色。

groups：组可以一组的用户，也可以将角色映射到角色中，用户可以成为组员后继承用组的角色

realms：领域，领域管理着一批，用户、证书、角色、组等，一个用户只能属于且能登陆到一个域，域之间是互相独立的，域只能管理在它下面的用户。

clients：客户端是一个实体，可以请求keycloak对用户进行身份验证，大部分情况下，客户端是应用或服务希望使用keycloak来保护自己和提供一个单点登录的解决方案。客户端也可以是一个实体，请求身份信息或一个访问信息，这样就可以调用其他keycloak保护的应用或服务了。

client adapters：

```
配置会有以下模块：
Realm 设置: 基本设置，包括是否允许注册，登陆界面，token有效期等
Clients: 被认可等客户端或服务，支持 openid-connect，saml 协议Clients 
ScopesRoles: 角色定义，分 realm 角色和 client 角色，（还有混合两种的角色），然后可以绑定给用户，另：你可以为每个client指定是role scope（是否仅返回client role）
Identity Providers: 第三方登陆配置，例如配置通过 github 登陆等User 
Federation: 用户联邦，用户可以从 LDAP/AD，Kerberos 等同步
authentication: 定义支持的登陆方式，包括登录的密码规则定义等
管理有以下模块：
groups: 组管理
users: 用户管理
Sesssions: session 管理
events: 事件管理
Import: 通过xml导入数据
Export: 导出数据
```

## 单点登录

Keycloak 是一个优秀的开源的单点登录工具。想使用系统 B的用户直接登录系统 A，而不用再次去系统 A 里注册，这也是典型的单点登录场景。单点登录有多种协议：

### 单点登录认证协议

最知名的单点登录认证协议主要是 OpenId Connect 和 SAML。下面就来看看认证服务器和被保护的应用是怎么和这些协议打交道的。

### OpenId Connect

OpenId Connect 通常简称为 OIDC，它是在 OAuth 2.0 的基础上扩展而成的认证协议。OAuth 2.0 只是一个构建认证协议的框架，并且很不完整，但是 OIDC 确实一个羽翼丰满的认证与授权协议。OIDC 严重使用 Json Web Token（JWT）标准。这些标准用紧凑和对网络友好的方式定义了 JSON 格式的唯一标记以及对数据进行数字化签名和加密的方法。

OIDC 在 Keycloak 中的使用场景分为两种类型。第一种是应用请求 Keycloak 服务器来认证用户。当成功登录后，应用会收到一个名称为 `access_token` 的唯一身份标志符。这个唯一身份标志符包含了诸如用户名、电子邮箱、以及其他个人资料等等信息。 `access_token` 会被 realm 进行数字签名，并且包含用户的可访问信息（比如用户-角色映射），从而应用可以使用它来决定该用户被允许访问应用中的哪些资源（上面的在线演示中显示的已登录用户没有权限页面，就是因为拿到的 `access_token` 中没有相关的可访问信息）。

第二种使用场景类型是客户端想获取远端服务的访问权限。在这个场景下，客户端请求 Keycloak 来获取一个访问令牌来代表用户调用远端服务。Keycloak 认证该用户后询问用户是否同意为该客户端授予访问权限。一旦用户同意授权，客户端就会收到访问令牌。这个访问令牌是由 realm 数字化签名过的。该客户端随后就可以使用这个访问令牌向远端服务发起 REST 调用了。这个 REST 服务抽取出访问令牌，验证令牌的签名，然后基于令牌中的可访问信息决定是否保护这个调用请求。

### OIDC 认证流程

OIDC 有多种不同的方式为客户端或者应用提供用户认证并接收身份标记和访问令牌。你要使用那种方式很大程度上取决于应用或者客户端请求访问权限的类型。所以 OIDC 的认证流程既和 Oauth2 类似又有区别，基本流程如下：

1. 客户端准备包含所需请求参数的身份验证请求。
2. 客户端将请求发送到授权服务器。
3. 授权服务器对终端用户进行身份验证。
4. 授权服务器获得终端用户同意/授权。
5. 授权服务器将 code 发送回客户端 。
6. 客户端将 code 发送到令牌端点获取 access_token 和 idToken。
7. 客户端使用私钥验证 idToken 拿到用户标识 or 将 access_token 发送到授权服务器获取用户标识。

这里注意最后第 6、7 点操作，这里开始 OIDC 和 Oauth2 不一样了

### 授权码流程

这是一个基于浏览器的协议，也是在验证和授权基于浏览器的应用时所推荐使用的。它严重依赖浏览器重定向来获取身份标记与访问令牌。总结如下：

1. 使用浏览器访问应用。这个应用会提醒用户当前还未登录，所以它指示浏览器重定向到 Keycloak 来认证。该应用会以查询参数的形式在浏览器重定向时向 Keycloak 传递一个回调 URL（即演示截图中的 redirect_uri），Keycloak 在完成认证后会使用到它。
2. Keycloak 认证用户，并创建一次性、非常短时间有效的临时码。Keycloak 通过前面提供的回调 URL 重定向回到应用，同时将临时码作为查询参数附加到回调 URL 上。
3. 应用抽取临时码，并且在后端通过不同于前端的网络渠道向 Keycloak 发起 REST 调用，使用临时码交换身份标记、访问令牌以及刷新令牌。一旦这个临时码在获取令牌中被使用过了，它就不能再次被使用了。这防止了潜在的重放攻击。

非常重要的一点是访问令牌通常有效期很短，一般在分钟级别过期。而刷新令牌由登录协议传送，允许应用在访问令牌过期后去获取一个新的访问令牌。这样一个刷新协议在受损系统中非常重要。如果访问令牌有效期很短，那么整个系统仅仅在被盗用的令牌剩余的有效期内是处于被攻击状态的。如果管理员吊销了访问权限，那么接下来的令牌刷新请求会失败。这样更加安全并且可伸缩性更好。

该流程另一个重要的方面是所谓的开放客户端还是保密客户端的概念。保密的客户端在使用临时码交换令牌时需要提供客户端密钥。开放客户端则不需要。只要严格使用 HTTPS 并且客户端的重定向 URI 被严格注册，那么采用开放客户端完全没有问题。由于无法使用安全的方式传输客户端密钥，所以 HTML5/JavaScript 客户端不得不天然就属于开放客户端。再次强调，这仅仅在严格使用 HTTPS 并严格注册重定向 URI 时是可以的。

### 隐式流程

这也是一个基于浏览器的协议，很类似授权码流程，只是请求量更少，也不需要刷新令牌。该流程不被推荐，因为存在访问令牌泄漏的可能性。比如由于令牌通过重定向 URI （详见下）传输，所以可能通过浏览器历史记录泄漏。并且，由于该流程没有为客户端提供刷新令牌的服务，所以访问令牌不得不设置一个更长的时间，不然当令牌失效后用户需要再次认证。Keycloak 不推荐这种流程但是仍然支持这种流程，因为它存在于 OIDC 和 OAuth 2.0 的规格文档中。总结如下：

1. 使用浏览器访问应用。应用提示用户当前还未登录，所以它指示浏览器重定向到 Keycloak 去认证。应用将回调 URL （一个重定向 URI）作为查询参数传递给 Keycloak，在认证完成后会被其使用。
2. Keycloak 认证用户并且创建身份标记和访问令牌。Keycloak 使用之前提供的回调 URL 重定向回到应用并使用查询参数的方式额外添加身份标记和访问令牌在回调 URL zhong。
3. 应用从回调 URL 中抽取身份标记和访问令牌。

### 资源拥有者密码凭据授权（直接访问授权）

这在 Keycloak 管理员控制台中指直接访问授权。当 REST 客户端希望代表用户获取令牌时使用该流程。这是一个 HTTP POST 请求，该请求中包含了用户的安全凭据和客户端 id，以及客户端的密钥（如果是保密客户端的话）。该用户的安全凭据随请求中的表单参数发送。这个 HTTP 响应中包含的身份标记、访问权限，以及刷新令牌。

### 客户端凭据授权

这也是由 REST 客户端使用的，但不是代表一个外部用户去获取令牌，而是基于和客户端相关的元数据与服务账号的权限来创建一个令牌。

# 操作

## 创建新的领域(realm)

Realm是一个隔离的概念，Realm A中的用户与Realm B中的用户完全隔离。当然，也可以不创建Realm，直接用 Master 这个Realm，不过一般来说，为了资源的隔离，以及更好的安全性不太建议与管理员共用Realm。从Master的下拉菜单中点击"Add Realm"，如下图所示：

![img](https://upload-images.jianshu.io/upload_images/23610677-6dc21454cd9201df)

## 创建新的用户

点击左边导航栏的Users，点击右边的"Add user"来创建新的用户。输入用户名，此参数是必须输入，其他字段可选。点击保存，会出现"Credentials"标签，输入用户名和密码，然后”reset password”,可以关闭临时密码功能。如下图，

![img](https://upload-images.jianshu.io/upload_images/23610677-d1e1ae9bdd3845cf)

接着登出Keycloak，使用上面创建的用户名和密码登陆创建的demo realm，http://localhost:8080/auth/realms/demo/account。

# docker安装gitlab-ce

```
docker run --name gitlab --hostname 192.168.2.45 -d -p 24:22 -p 80:80 -p 433:433 gitlab/gitlab-ce
```

初始的密码在以下文件中，登录后**尽快修改密码**，该文件会在24小时内删除

```
/etc/gitlab/initial_root_password
```

![image-20220524174938305](http://myimg.go2flare.xyz/img/image-20220524174938305.png)

![gitlab藏文件](https://img-blog.csdnimg.cn/971a679ce8854faeaff393173898694a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RpbW9uaXVt,size_16,color_FFFFFF,t_70)

## docker安装keycloak

```
docker run -dp 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=changeme quay.io/keycloak/keycloak:18.0.0 start-dev
```

# 实例

## 微信登陆

### REF

> [使用 OIDC 在一个 Keycloak 中集成另一个 Keycloak 用户认证 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/360846976)

### 问题重述

我现在有2个系统a和b。他们都接入了keycloak，是2个realm。数据库也是隔离的。现在想要实现a系统通过b系统的账号登录。我想用类似微信登录的方式，把b系统当成微信。不知道这样是否合适，以及具体的实现感觉有点困难。

解答：

把b系统当成微信，实际上就是把b系统当成a系统的一个idp，即b系统是a系统的身份认证提供商。可以在a系统添加一个idp，在配置时把b系统配置进去就行了

### 实现

## gitlab使用SAMl协议

### REF

> [在自托管 GitLab 实例中集成 Keycloak 登录 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/405425214)

### 配置 KeyCloak

接下来，需要在 Keycloak 中配置 GitLab 客户端，因为我们的目的是在 GitLab 中启用 Keycloak 登录，那么 Keycloak 就是一个 IDP 了，而 GitLab 是其一个 OAuth 2 客户端应用。要将 Keycloak 当作 IDP，一般有两种形式，OIDC 形式和 SAML 形式。 

使用 OIDC 的形式集成，有两篇文章做过详细的介绍： 

- 《[使用 OIDC 在一个 Keycloak 中集成另一个 Keycloak 用户认证](https://zhuanlan.zhihu.com/p/360846976)》
- 《[在 eggjs 中集成 Keycloak 用户认证](https://zhuanlan.zhihu.com/p/359057316)》

如果对这种集成方式感兴趣，可以详细参考。这里准备**采用 SAML 方式**集成，以便补充这种集成方式的实战。实际上，网上已经有一些 GitLab 和 Keycloak 采用 SAML 集成的文章了，但是采用了非常复杂的方法，因此整篇文章都在讲集成细节。这里采用了一个**简单**的方式，只需要一小节介绍即可。 



### 导出 GitLab 的 SAML 元数据 xml 定义

- 这里卡住了，不知道怎么导出saml元数据

对于支持 SAML 集成的应用，可以访问其 `/users/auth/saml/metadata` 获得其 SAML 源数据信息。将其保存到本地： 

![img](https://pic1.zhimg.com/80/v2-d3ac3a85274e69385917c9589c194ec4_1440w.jpg)

### 在 Keycloak 中添加 GitLab 客户端的 email 映射

你可以添加更多的映射，以便把 Keycloak 中的用户信息映射到 GitLab 系统。但是只有 email 是必须的。 



![img](https://pic1.zhimg.com/80/v2-0bb425c1ff40b389b9253538fbb26b10_1440w.jpg)



### 配置 GitLab



现在，Keycloak 中已经有了我们新部署的 GitLab 实例信息，现在需要配置 GitLab，让它知道我们部署好的 Keycloak 实例信息。 

首先，需要记下 Keycloak 应用对应的 Realm 的 url，以及拷贝一下它的证书信息： 



![img](https://pic2.zhimg.com/80/v2-8a787ee095d05c900703a1cf36bef3b9_1440w.jpg)



然后，回到 CodeSpaces，进入到 GitLab docker 镜像里： 

## gitlab使用OIDC协议

## 背景：

最近收到客户需要我们 dx-arch 系统对接他们的 sso 系统， 他们的 sso 系统大多都是基于oauth2 写的。但是keycloak 在对接 IDP 的协议只声明了支持OIDC 和 SAML 协议，

关于OIDC 之前已经有文档测试过了对接，怀着测试的心情我打算找一个支持oauth2 系统的试一试keycloak 是否可以通过oauth2 对接第三方系统。 但是找了许久的开源包没有找到合适的oauth2 server ，

gitlab 也是通过 oauth2 flow 支持sso的， 因此就打算使用gitlab来试一试水





## 搭建gitlab：

由于本次测试的重点在keycloak 和gitlab oauth2 的对接， 所以这里gitlab 部署就是简单通过docker 进行部署。

**部署gitlab** 展开源码

等容器启动之后需要**等待几分钟**让gitlab 跑起来。（不要问我为啥加粗标注😂， 因为我之前老是以为是我启动姿势有问题。 看了半天日志没报错，然后才起来的， gitlab在容器内做了一系列设置所以启动的有点慢，如果机器卡的话可能会更慢一点）



## 配置gitlab

打开 [http://10.6.165.11](http://10.6.165.11/) 即可看到设置密码的页面， 默认的超级管理员的用户名是 root 这里设置的密码就是root的密码

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-18-4.png?version=1&modificationDate=1619579179373&api=v2)

设置完密码之后会要求登陆，使用用户名 root 以及刚刚设置的密码 登陆

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-18-41.png?version=1&modificationDate=1619579179253&api=v2)

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-20-35.png?version=1&modificationDate=1619579179037&api=v2)

点击创建application

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-22-30.png?version=1&modificationDate=1619579178874&api=v2)

填写keycloak application 相关参数

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-25-45.png?version=1&modificationDate=1619579178610&api=v2)

获取application 的ID 和secret 

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-26-47.png?version=1&modificationDate=1619579178460&api=v2)





## 配置keycloak

在keycloak 中创建IDP（identity Providers）， 这里选择OIDC 的原因是没有keycloak没有提供oauth2的协议选项， OIDC的本质流程和oauth很像

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-28-35.png?version=1&modificationDate=1619579178253&api=v2)

创建IDP

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-32-52.png?version=1&modificationDate=1619579177572&api=v2)

设置keycloak 的登陆设置， 隐藏其自身的登陆页面。

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-35-34.png?version=1&modificationDate=1619579177431&api=v2)

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-36-21.png?version=1&modificationDate=1619579177232&api=v2)





## 测试登陆：

当我通过dx-arch 跳转打开keycloak登陆页面时就会直接跳转到gilab的登陆页面

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-47-32.png?version=1&modificationDate=1619579176973&api=v2)

当检索到当前keycloak中已经存在有相关信息冲突的用户或者不存在当前的gitlab用户时会跳转让你更新用户信息， keycloak会重新在keycloak中创建用户并入库，

且当gitlab用户数据变更时，用户下次登陆是不会更新keycloak中的用户数据和gitlab保持一致， 需要手动在keycloak中更改用户信息。

![img](https://dwiki.daocloud.io/download/attachments/84640566/image2020-9-15_17-51-3.png?version=1&modificationDate=1619579176720&api=v2)





## 总结：

1. keycloak 通过OIDC 对接第三方需要填写如下相关参数：
   1. client ID
   2. client Secret
   3. auth URL
   4. token URL
2. 以上参数均是oauth2中相关的参数， OIDC 协议是扩充了Oauth2 的一些参数，本质上还是走的Oauth2的流程， 因此理论上keycloak 可以对接实现了oauth2 的第三方sso server。 并且通过可以对接gitlab 也可以佐证这一点。











![image-20220525173012126](http://myimg.go2flare.xyz/img/image-20220525173012126.png)

![image-20220525173033989](http://myimg.go2flare.xyz/img/image-20220525173033989.png)

![image-20220525173049173](http://myimg.go2flare.xyz/img/image-20220525173049173.png)

![image-20220525173122620](http://myimg.go2flare.xyz/img/image-20220525173122620.png)

![image-20220525173357046](http://myimg.go2flare.xyz/img/image-20220525173357046.png)

![image-20220525173412834](http://myimg.go2flare.xyz/img/image-20220525173412834.png)







```

docker exec -it 容器名 /bin/bash
vi /etc/gitlab/gitlab.rb

```







## keycloak做gitlab sso单点登录

### 首先部署gitlab--ee实例

gitlab有两个版本，一个是Gitlab-ce（Gitlab Community Edition），一个是Gitlab-ee（Gitlab Enterprise Edition）

本文使用docker-compose的方式安装gitlab-ee

- windows

```
services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: 'localhost'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://localhost:8929' 
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - '8929:8929'
      - '2224:22'
    volumes:
      - '/c/doc_repo/gitlab-ee/etc/gitlab:/etc/gitlab'
      - '/c/doc_repo/gitlab-ee/var/log/gitlab:/var/log/gitlab'
      - '/c/doc_repo/gitlab-ee/var/opt/gitlab:/var/opt/gitlab'
```

- linux

```
services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: '10.29.3.30'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://10.29.3.30:8929' 
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - '8929:8929'
      - '2224:22'
    volumes:
      - '/usr/local/pro_repo/doc_repo/gitlab-ee/etc/gitlab:/etc/gitlab'
      - '/usr/local/pro_repo/doc_repo/gitlab-ee/var/log/gitlab:/var/log/gitlab'
      - '/usr/local/pro_repo/doc_repo/gitlab-ee/var/opt/gitlab:/var/opt/gitlab'
```

> hostname需要替换成可使用的endpoint，如本地的网卡ip等
>
> 挂载的文件目录需要根据需要自行调整

```
docker compose up -d 
```

![image-20220527175722832](http://myimg.go2flare.xyz/img/image-20220527175722832.png)

看到容器如下状态，如果是starting状态则需要等待几分钟

```
health：starting（正在启动容器，需耐心等待几分钟）

healthy（容器已启动）
```

![image-20220527175938429](http://myimg.go2flare.xyz/img/image-20220527175938429.png)

![image-20220527175815584](http://myimg.go2flare.xyz/img/image-20220527175815584.png)

容器启动后，按配置中的endpoint（10.29.3.30:8929），即可访问容器实例的服务

首次登录需要用户的密码

![image-20220527180140434](http://myimg.go2flare.xyz/img/image-20220527180140434.png)

初始密码在容器实例的文件中，进入刚部署的容器中，查看初始密码并复制，登录gitlab后**尽快修改密码**，该文件会在24小时内删除

```
# 进入容器
docker exec -it gitlab-ee-web-1 /bin/bash
# 查看初始密码
cat /etc/gitlab/initial_root_password
```

![image-20220526112242428](http://myimg.go2flare.xyz/img/image-20220526112242428.png)

![image-20220526112409281](http://myimg.go2flare.xyz/img/image-20220526112409281.png)

### 配置Keycloak

- client设置

```
clients -> [gitlab] -> settings
```

![VeryCapture_20220529112637](http://myimg.go2flare.xyz/img/VeryCapture_20220529112637.jpg)

> Valid Redirect URLs：gitlab重定向的地址
>
> BaseURL：keycloak中可跳转的client的地址
>
> Master SAML Processing URL：供client Post （sign out等）的地址？
>
> IDP initiated SSO URL Name：初始化client的重定向地址名称，最好和client ID一致

- client mapper

映射器将用户信息映射到 GitLab 的 SAML 2.0请求中的参数。例如，一个email映射器将keycloak中的email映射到 Gitlab中，在gitlab只有email是必须的

```
clients -> [gitlab] -> settings -> mappers
```

![image-20220527174123758](http://myimg.go2flare.xyz/img/image-20220527174123758.png)

![image-20220527174056310](http://myimg.go2flare.xyz/img/image-20220527174056310.png)

### 配置GItlab

现在keycloak中已经配置有Gitlab的信息，我们需要在Gitlab中配置Keycloak实例信息，首先进入Keycloak中对应Gitlab的Realm Setting，复制其证书信息

![image-20220526191439690](http://myimg.go2flare.xyz/img/image-20220526191439690.png)

打开宿主机目录中的gitlab.rb文件

```
C:\doc_repo\gitlab-ee\etc\gitlab\gitlab.rb
```

![image-20220526115907402](http://myimg.go2flare.xyz/img/image-20220526115907402.png)

在gitlab.rb文件中配置keycloak有关信息，配置信息如下

```
gitlab_rails['omniauth_enabled'] = true
gitlab_rails['omniauth_allow_single_sign_on'] = ['saml']
# gitlab_rails['omniauth_auto_sign_in_with_provider'] = 'saml'
gitlab_rails['omniauth_block_auto_created_users'] = false
gitlab_rails['omniauth_auto_link_saml_user'] = true
gitlab_rails['omniauth_providers'] = [
    {
        name: 'saml',
        args: {
            assertion_consumer_service_url: '<gitlab回调地址>/',
            idp_cert: " -----BEGIN CERTIFICATE-----
\n<YOUR CERTIFICATE>\n -----END CERTIFICATE----- \n",
            idp_sso_target_url: '<keycloak重定向地址>',
            issuer: '<gitlab>',
            name_identifier_format: 'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent'
        },
        label: 'KEYCLOAK 登录'
    }
]
```

可以直接在gitlab.rb文件中搜索**OmniAuth Settings配置块**，按需修改配置块中的内容，也可以直接在**文件末尾附加**配置信息

> gitlab_rails['omniauth_auto_sign_in_with_provider'] = 'saml'：如果不需要gitlab的登录界面，默认用DX-ARCH的话加上该选项
>
> gitlab_rails['omniauth_block_auto_created_users'] = false：不允许自动创建用户，选择false即可以通过keycloak自动创建gitlab用户
>
> gitlab_rails['omniauth_auto_link_saml_user'] = true：keycloak user自动链接到gitlab已有用户
>
> assertion_consumer_service_url：gitlab的回调地址，一般都为固定/users/auth/saml/callback路径，url前面更改为你的gitlab地址
>
> idp_cert：keycloak中realm的证书，即上一步中复制的证书信息
>
> idp_sso_target_url：重定向keycloak地址，将realm替换为你设置client的realm，client替换为设置的clientID
>
> issuer：keycloak中设置的Client ID

实例

```

gitlab_rails['omniauth_enabled'] = true
gitlab_rails['omniauth_allow_single_sign_on'] = ['saml']
# gitlab_rails['omniauth_auto_sign_in_with_provider'] = 'saml'
gitlab_rails['omniauth_block_auto_created_users'] = false
gitlab_rails['omniauth_auto_link_saml_user'] = true
gitlab_rails['omniauth_providers'] = [
    {
        name: 'saml',
        args: {
            assertion_consumer_service_url: 'http://10.29.3.30:8929/users/auth/saml/callback',
            idp_cert: " -----BEGIN CERTIFICATE-----
\nMIIClzCCAX8CBgGA8IXOwjANBgkqhkiG9w0BAQsFADAPMQ0wCwYDVQQDDAR0ZXN0MB4XDTIyMDUyMzEwNDQwMloXDTMyMDUyMzEwNDU0MlowDzENMAsGA1UEAwwEdGVzdDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAIAmQyDT06QqwNQzw24yGrOo0krHmxhWDOfoXbW+VskfhRcpdNbh3Q1y6hVfsEY50JhWcHPX3NUzG4xkztJ7JogtRDy082qAcR4Teg0Vj8iRV9tkU0SUAgxff+r2o3RErwy1IfONqHLLdGA8odScCX9jsa30IoIi9bujCwftUTvRux8JORL8HWSxxk2KyGUGsA1gcGa5TFiszetQBFFSWliMwt9xc2dv2Ss64v1866wrGfYvBjrq60KcCoNsJwCgQkUzl1ODHFD3YYNzSW0OprIqUlBaC4DVRXzyMiev99JwKhxgWb+e9naoDaGCU7WAMJ53KrLZx2U9OzWyNWA89RECAwEAATANBgkqhkiG9w0BAQsFAAOCAQEALr94VGV2ZP6e7TNeBYyEvdkxM0izzRTvgfUY9wwfQiIhSRKZMggdQ6GHqmxQxNWT/R5W+1j4nWDENFYFBcByA/SsXUM/xfpPcWVM2523BCqRs0Nlp6AIM06JT0WcFoKoM0cKNeEuxvGDQf8rffkfRDfI2DVw5jzvqVzqrYO+pEr1LlXqMXyLSTIcOt0qYIxdzCt50Bn0YToooYLCT+eFqU0JUpFI7E5NPybijGipUDxZt5lSM8S6KYEFQgwHG0fB5wIX6iiz3CI3bRETxe8DwEwSD79GMGZEzeSZKAqlk/ey8oAkFJV9opOxdkrW6+va1DJvQVE0VQU/CYJo/j9I9g==\n -----END CERTIFICATE----- \n",
            idp_sso_target_url: 'http://10.29.3.30:30035/auth/realms/test/protocol/saml/clients/http%3A%2F%2F10.29.3.30%3A8929',
            issuer: 'http://10.29.3.30:8929',
            name_identifier_format: 'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent'
        },
        label: 'DX-ARCH-TEST'
    }
]
```

修改完后  **: wq** 保存后退出配置文件，重启Gitlab容器或者使用以下命令重新配置Gitlab

```
gitlab-ctl reconfigure
```

就可以在gitlab中看到Keycloak的登录入口了

![image-20220526191831794](http://myimg.go2flare.xyz/img/image-20220526191831794.png)

点进DX-ARCH

![image-20220527173508331](http://myimg.go2flare.xyz/img/image-20220527173508331.png)



> REF:
>
> [在自托管 GitLab 实例中集成 Keycloak 登录 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/405425214)
>
> [medium.com(需翻墙)](https://dheeruthedeployer.medium.com/gitlab-integration-with-keycloak-e1b2ff11a177#:~:text=STEP Create a SAML Client in Keycloak.,Replace the gitlab.server.com with your GitLab server name.)
>
> [ChathuminaVimukthi/Gitlab-SSO-implementation-using-Keycloak: Gitlab SSO implementation using Keycloak (github.com)](https://github.com/ChathuminaVimukthi/Gitlab-SSO-implementation-using-Keycloak)
>
> [gitlab统一登录与跳转 - 广州项目管理 - DaoCloud 知识库](https://dwiki.daocloud.io/pages/viewpage.action?pageId=121164087)

### 常见问题

## gitlab重定向到keycloak显示page not found

![image2022-5-29_15-45-30](http://myimg.go2flare.xyz/img/image2022-5-29_15-45-30.png)

### 解决方式

**keycloak版本较新**，在gitlab.rb中设置keycloak的重定向地址中把auth去掉

![image2022-5-29_15-47-39](http://myimg.go2flare.xyz/img/image2022-5-29_15-47-39.png)

## gitlab重定向到keycloak显示client not found

![image2022-5-29_15-54-46](http://myimg.go2flare.xyz/img/image2022-5-29_15-54-46.png)

### 解决方式

在keycloak和gitlab的配置中确定以下几个位置都**设置为keycloak 的client ID**

![image2022-5-29_15-56-22](http://myimg.go2flare.xyz/img/image2022-5-29_15-56-22.png)



## keycloak做gitlab sso单点登录使用OCID协议

该方式需要keycloak采用http2部署，且gitlab中包含有keycloak的证书







> REF:
>
> [OpenID Connect OmniAuth provider |GitLab](https://docs.gitlab.com/ee/administration/auth/oidc.html#keycloak)
>
> [keycloak集成gitlab - 北方姆Q - 博客园 (cnblogs.com)](https://www.cnblogs.com/bfmq/p/15917975.html)

![image-20220526140339746](http://myimg.go2flare.xyz/img/image-20220526140339746.png)

这个是最狗的一个问题，但不是都会出现，这个是https证书的问题。你的keycloak必须配置成https的并且gitlab的服务器认这个证书，所以如果是自签证书就麻烦了点，建议keycloak跟gitlab甚至所有服务都配好**合法**证书，会在无形间解决无数麻烦

容器里的gitlab加ca证书  cat /etc/gitlab/rootCA.pem >> /opt/gitlab/embedded/ssl/certs/cacert.pem && update-ca-certificates 

![img](https://img2022.cnblogs.com/blog/996591/202202/996591-20220221105233882-1418684821.png)

 

# K8S部署gitlab

## 创建nfs共享服务器

通过yum的方式进行安装

```
# 安装nfs服务
yum install nfs-utils
 
# 安装rpc服务
yum install rpcbind
 
# 关于开机自启动参考实际情况进行配置，注意防火墙和iptable的限制
```

*比如这里我想直接将整个data用于存储，那我可以直接使用就地使用，但注意权限，修改权限使用 `chmod 755`*

![img](https://dwiki.daocloud.io/download/attachments/103946596/image2021-12-4_2-14-21.png?version=1&modificationDate=1638555261546&api=v2)

*如果是新盘则需要进行格式化和挂载*

```
# 假设有块盘为sdc，需要先进行格式化处理
mkfs -t xfs /dev/sdc
 
# 挂载到目标文件夹中
mount /dev/sdc /data
```



## 2、修改文件 /etc/exports

```
#创建要共享的目录
mkdir /data/redis -p
#设置文件夹权限否则可能导致PVC创建失败
chmod -R 777 /data/redis

#编辑NFS配置并加入以下内容
vim /etc/exports
/data * (rw,sync,no_root_squash)
/data/postgresql/data * (rw,sync,no_root_squash)
/data/var/opt/gitlab/git-data * (rw,sync,no_root_squash)
/data/etc/gitlab * (rw,sync,no_root_squash)
/data/var/opt/gitlab/gitlab-rails/uploads * (rw,sync,no_root_squash)
/data/var/opt/gitlab/gitlab-rails/shared * (rw,sync,no_root_squash)
/data/var/opt/gitlab/gitlab-ci/builds * (rw,sync,no_root_squash)
/data/var/opt/gitlab * (rw,sync,no_root_squash)

#载入配置
exportfs -rv
```

*因为使用的是data目录，同时放行所有的IP（这个\*号就是所有IP，如果我要过滤192，就可以写成192.\*）*

*并且规则要可读写（rw），频繁通信（sync同步更新），no_root_squash(网页摘抄：登入 NFS 主机使用分享目录的使用者，如果是 root 的话，那么对于这个分享的目录来说，他就具有 root 的权限）*

![img](https://dwiki.daocloud.io/download/attachments/103946596/image2021-12-4_2-16-27.png?version=1&modificationDate=1638555387623&api=v2)



## 3、启动必要服务

```
# 先启动rpcbind
systemctl start rpcbind
 
# 再启动nfs
systemctl start nfs-server
```

![img](https://dwiki.daocloud.io/download/attachments/103946596/image2021-12-4_2-26-28.png?version=1&modificationDate=1638555988942&api=v2)

启动之后可以用一些简单命令查看配置文件是否成功:

```
# NFS服务器自己查看
exportfs
 
# 其他主机访问该NFS查看
showmount -e IP
```

![img](https://dwiki.daocloud.io/download/attachments/103946596/image2021-12-4_2-28-39.png?version=1&modificationDate=1638556120078&api=v2)

当出现上面的结果时表示NFS服务器已经健康运行

## 创建pv

创建PVC之后，应用就可以使用存储了，目录的挂载都可以在Pod的编排中声明。

PV(Persistent Volume)是对底层存储的一层抽象，他将不同形式的存储抽象为统一的资源，按需分配。同时，他将存储中的一些关键信息（如密码）统一管理起来，使用存储的应用只需向其申请即可。无需填写这些私密信息。

在DCE中创建应用模板，填写以下编排，视情况，修改存储大小、挂载目录，NFS服务器ip。创建之后部署即可

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitlab-nfs
  labels:
    type: gitlab-nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany # 多实例读写
  nfs:
    path: /nfs-data/gitlab-data
    server: 10.29.0.122 # NFS Server
  persistentVolumeReclaimPolicy: Retain
  

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-claim
  labels:
    app: gitlab
spec:
  accessModes:
    - ReadWriteMany
  selector:
    matchLabels:
      type: gitlab-nfs
  resources:
    requests:
      storage: 10Gi
```

> nfs server不知道怎么搞？看dce上别人的server就行呀

使用storage class方式自动分配存储nfs，集群中本身有声明好的storageclass，直接用即可

```
---
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



## 创建pvc

PVC（Persistent Volume Chaim）是用户存储的请求。 它类似于Pod。Pod消耗节点资源，PVC消耗存储资源。 Pod可以请求特定级别的资源（CPU和内存）。 权限要求可以请求特定的大小和访问模式。

在创建PV的基础上，创建PVC分配给不同应用使用。在DCE页面上创建应用模板，填写以下编排，视情况修改存储大小，之后直接部署即可

```

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-claim
  labels:
    app: gitlab
spec:
  accessModes:
    - ReadWriteMany
  selector:
    matchLabels:
      type: gitlab-nfs
  resources:
    requests:
      storage: 950Gi
```



## redis

在DCE页面创建应用模板，填写以下编排，之后直接点击部署到指定租户即可。

```
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
        - image: 10.29.3.30:30004/gitlab/redis:4.0.9-1
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



## Postgresql

在DCE页面创建应用模板，填写以下编排，之后直接点击部署到指定租户即可。

```
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
        - image: 10.29.3.30:30004/gitlab/postgresql:10
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

> 登录：psql -U postgres -h localhost
>
> 创建角色： CREATE ROLE gitlab WITH LOGIN PASSWORD 'gitlab' SUPERUSER; # 创建名为 `gitlab` 密码为gitlab的role
>
> 查看角色：select rolname,rolsuper,rolcanlogin from pg_roles;
>
> 创建数据库：create database gitlabhq_production;
>
> 查看数据库：\list
>
> 退出数据库：\q
>

## GitLab Application Server

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: gitlab-cm
data:
  gitlab: |
    nginx['enable'] = false
    gitlab_rails['trusted_proxies'] = ['10.29.3.30']
    gitlab_workhorse['listen_network'] = "tcp"
    gitlab_workhorse['listen_addr'] = "0.0.0.0:9229"
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
    gitlab_rails['gitlab_shell_ssh_port'] = 30022
    external_url 'http://10.29.3.30:10080'
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
                assertion_consumer_service_url: 'http://10.29.3.30:10080/users/auth/saml/callback',
                idp_cert: " -----BEGIN CERTIFICATE-----
    \nMIIClzCCAX8CBgGA8IXOwjANBgkqhkiG9w0BAQsFADAPMQ0wCwYDVQQDDAR0ZXN0MB4XDTIyMDUyMzEwNDQwMloXDTMyMDUyMzEwNDU0MlowDzENMAsGA1UEAwwEdGVzdDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAIAmQyDT06QqwNQzw24yGrOo0krHmxhWDOfoXbW+VskfhRcpdNbh3Q1y6hVfsEY50JhWcHPX3NUzG4xkztJ7JogtRDy082qAcR4Teg0Vj8iRV9tkU0SUAgxff+r2o3RErwy1IfONqHLLdGA8odScCX9jsa30IoIi9bujCwftUTvRux8JORL8HWSxxk2KyGUGsA1gcGa5TFiszetQBFFSWliMwt9xc2dv2Ss64v1866wrGfYvBjrq60KcCoNsJwCgQkUzl1ODHFD3YYNzSW0OprIqUlBaC4DVRXzyMiev99JwKhxgWb+e9naoDaGCU7WAMJ53KrLZx2U9OzWyNWA89RECAwEAATANBgkqhkiG9w0BAQsFAAOCAQEALr94VGV2ZP6e7TNeBYyEvdkxM0izzRTvgfUY9wwfQiIhSRKZMggdQ6GHqmxQxNWT/R5W+1j4nWDENFYFBcByA/SsXUM/xfpPcWVM2523BCqRs0Nlp6AIM06JT0WcFoKoM0cKNeEuxvGDQf8rffkfRDfI2DVw5jzvqVzqrYO+pEr1LlXqMXyLSTIcOt0qYIxdzCt50Bn0YToooYLCT+eFqU0JUpFI7E5NPybijGipUDxZt5lSM8S6KYEFQgwHG0fB5wIX6iiz3CI3bRETxe8DwEwSD79GMGZEzeSZKAqlk/ey8oAkFJV9opOxdkrW6+va1DJvQVE0VQU/CYJo/j9I9g==\n -----END CERTIFICATE----- \n",
                     idp_sso_target_url: 'http://10.29.3.30:30035/auth/realms/test/protocol/saml/clients/gitlab',
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
  ports:
    - name: gitlab-ui
      port: 9229
    - name: gitlab-ssh
      port: 22
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
            - containerPort: 9229
              name: gitlab-ui
            - containerPort: 30022
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



### 配置文件

```
apiVersion: apps/v1
kind: ConfigMap
metadata:
  name: gitlab-cm
data:
  gitlab: |
    nginx['enable'] = false
    gitlab_rails['trusted_proxies'] = ['172.25.254.5']
    gitlab_workhorse['listen_network'] = "tcp"
    gitlab_workhorse['listen_addr'] = "0.0.0.0:9229"
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
    gitlab_rails['gitlab_shell_ssh_port'] = 30022
    external_url 'http://172.25.254.5:10080'
    user['uid'] = 9000
    user['gid'] = 9000
    web_server['uid'] = 9001
    web_server['gid'] = 9001
    registry['uid'] = 9002
    registry['gid'] = 9002
    gitlab_rails['omniauth_allow_single_sign_on'] = ['github']
    gitlab_rails['omniauth_providers'] = [
      {
        "name" => "github",
        "app_id" => "ea4146f6f57b1011d3d5",
        "app_secret" => "b3eeea887c5f37460b0db1da4eaa88652288d2ec",
        "url" => "http://git.gtmc.com.cn/",
        "args" => { "scope" => "user:email" }
      }
    ]
```



### 初始化Job

在DCE中创建应用模板，填写以下编排，视情况，修改外部nginx地址。创建之后部署即可

```
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
              cpu: 0.5
              memory: 0.5Gi
            requests:
              cpu: 0.5
              memory: 0.5Gi
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

```
+    # Trusted Proxies
      +    # Customize if you have GitLab behind a reverse proxy which is running on a different machine.
      +    # Add the IP address for your reverse proxy to the list, otherwise users will appear signed in from that address.
      如果您在另一台机器上运行的反向代理后面有 GitLab，请自定义。将反向代理的 IP 地址添加到列表中，否则用户将显示从该地址登录。

```



### Deployment

在DCE中创建应用模板，填写以下编排，视情况，修改外部nginx地址。创建之后部署即可，完成GitLab应用的部署。

```
---
apiVersion: v1
kind: Service
metadata:
  name: gitlab
  labels:
    app: gitlab
spec:
  ports:
    - name: gitlab-ui
      port: 9229
    - name: gitlab-ssh
      port: 22
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
  replicas: 2
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
        - image: 192.168.110.253/gitlab/gitlab-ce:12.4.1-ce.0
          name: gitlab
          resources:
            limits:
              cpu: 4
              memory: 16Gi
            requests:
              cpu: 4
              memory: 16Gi
          env:
            - name: GITLAB_OMNIBUS_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: gitlab-cm
                  key: gitlab
          ports:
            - containerPort: 9229
              name: gitlab-ui
            - containerPort: 30022
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

```
这里的external_url指的是：gitlab中项目里，clone 项目地址时前缀所使用的url
```

![image-20220601005002766](http://myimg.go2flare.xyz/img/image-20220601005002766.png)

![image-20220601005246387](http://myimg.go2flare.xyz/img/image-20220601005246387.png)

# 常见问题

## 遇到的问题

gitlab的日志中有重要的内容： You are using PostgreSQL 10.4 for the main database, but PostgreSQL >= 12
  is required for this version of GitLab. 提醒了postgreSQL版本过低，需要更新下版本到12以上。

```
Running handlers:

Notes:
Default admin account has been configured with following details:
Username: root
Password: You didn't opt-in to print initial root password to STDOUT.
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.

NOTE: Because these credentials might be present in your log files in plain text, it is highly recommended to reset the password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Running handlers complete
There was an error running gitlab-ctl reconfigure:

rails_migration[gitlab-rails] (gitlab::database_migrations line 51) had an error: Mixlib::ShellOut::ShellCommandFailed: bash[migrate g 	tlab-rails database] (/opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/resources/rails_migration.rb line 16) had an error: Mixlib::ShellOut::ShellCommandFailed: Expected process to exit with [0], but received '1'
---- Begin output of "bash"  "/tmp/chef-script20220601-27-1nqtzjm" ----
STDOUT: ██     ██  █████  ██████  ███    ██ ██ ███    ██  ██████ 
          ██     ██ ██   ██ ██   ██ ████   ██ ██ ████   ██ ██      
          ██  █  ██ ███████ ██████  ██ ██  ██ ██ ██ ██  ██ ██   ███ 
          ██ ███ ██ ██   ██ ██   ██ ██  ██ ██ ██ ██  ██ ██ ██    ██ 
           ███ ███  ██   ██ ██   ██ ██   ████ ██ ██   ████  ██████  

******************************************************************************
  You are using PostgreSQL 10.4 for the main database, but PostgreSQL >= 12
  is required for this version of GitLab.
  
  Please upgrade your environment to a supported PostgreSQL version, see
  https://docs.gitlab.com/ee/install/requirements.html#database for details.
******************************************************************************
psql:/opt/gitlab/embedded/service/gitlab-rails/db/structure.sql:203: ERROR:  unrecognized partitioning strategy "hash"
rake aborted!
failed to execute:
psql --set ON_ERROR_STOP=1 --quiet --no-psqlrc --file /opt/gitlab/embedded/service/gitlab-rails/db/structure.sql --single-transaction gitlabhq_production

Please check the output above for any errors and make sure that `psql` is installed in your PATH and has proper permissions.

/opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/database/postgresql_database_tasks/load_schema_versions_mixin.rb:10:in `structure_load'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/db.rake:67:in `block (3 levels) in <top (required)>'
/opt/gitlab/embedded/bin/bundle:23:in `load'
/opt/gitlab/embedded/bin/bundle:23:in `<main>'
Tasks: TOP => db:schema:load
(See full trace by running task with --trace)
STDERR: 
---- End output of "bash"  "/tmp/chef-script20220601-27-1nqtzjm" ----
Ran "bash"  "/tmp/chef-script20220601-27-1nqtzjm" returned 1

Chef Infra Client failed. 140 resources updated in 01 minutes 48 seconds

Notes:
Default admin account has been configured with following details:
Username: root
Password: You didn't opt-in to print initial root password to STDOUT.
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.

NOTE: Because these credentials might be present in your log files in plain text, it is highly recommended to reset the password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.
```

![image-20220601114715953](http://myimg.go2flare.xyz/img/image-20220601114715953.png)

![image-20220601114730996](http://myimg.go2flare.xyz/img/image-20220601114730996.png)

![image-20220601142718449](http://myimg.go2flare.xyz/img/image-20220601142718449.png)

```
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: GSdr8GqMTeFkob3To7I5UaZx+bXmE9hINj3UMuWCg9w=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.++
```

## 初始密码无法登录

重置gitlab的登录密码步骤：
1、获取容器的id或者别名

```shell
#容器部署
docker ps

#集群部署
k get pod -n gitlab
```

2、进入容器

```shell
#gitlab为一开始设置的容器别名，也可以使用容器id
#容器部署
docker exec -it gitlab bash

#集群部署
k exec -it gitlab -n gitlab -- bash
```

3、启动Rails控制台

```shell
gitlab-rails console -e production
```



等待执行完，会进入输入模式
4、获取用户，设置密码

```shell
//第一个默认为root
user = User.where(id: 1).first
//必须同时更改密码和password_confirmation才能使其正常工作
user.password = '新的密码'
user.password_confirmation = '新的密码'
```

5、保存

```shell
//保存，稍等一会就会执行刚才输入的代码
user.save!
```


6、退出容器

```shell
ctrl+d
```

![image-20220608171306546](http://myimg.go2flare.xyz/img/image-20220608171306546.png)

然后就可以使用刚才输入的密码登录root账号了
