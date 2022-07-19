# 用户鉴权cookie-session，jwt

httpi协议是无状态，无连接。多个请求之间是无关联的，独立的

登录一个网站，搜索商品，下订单，支付，评论



## Cookie

什么是cookie？

cookie是在服务器产生的一小段文本信息，格式是字典，键值对

cookie的分类

会话级：保存内存，浏览器断开消失

持久化：保存硬盘，只有失效时间到了会被清除

**cookie如何实现鉴权**

当客户端第一次访问服务器时，服务端就会产生Cookie，然后通过响应头的方式在set-cookie中传输到客户端，客户端从第2-N次请求都会自动带上Cookie



弱点：cookie保存在客户端，对于一些敏感信息，用户名，密码，身份证等等会有安全问题，一旦泄露了，信息也就泄露

## Session

浏览器和服务器是在进行会话的

浏览器第一次访问服务器就是会话的开始，比较模糊的是会话结束的时间，所以每个网站的服务器都会给每个用户设置一个sessionID+过期时间，因为是服务器定义的，一般是存储在数据库里，sessionID是一段无规律的字符串，不保存用户的真实敏感信息，且服务端会cookie进行签名，即使被黑客截获修改了sessionID，那服务器就无法识别该sessionID，所以浏览器保存cookie后，浏览器第2-n次都会带上这个sessionID，直到浏览器中的cookie有效期过期后，浏览器就会自行删除cookie，也就是会话结束了。

打个比方，每个人都有身份证，去政府部门办事时都出示身份证，工作人员会核验验证身份后，才会给办理具体的业务，sessionID就是身份证，cookie就是你这个人，办事部门就是后台接口，核验就是接口在比对sessionID，用户没有登陆的时候有无sessionID？有。相当于刚出生时就没有身份证，而有医院的出生证明。且session-cookie是有有效期的，有效期由程序员根据业务需求决定。



## cookie-session机制

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650304008664-92639aaf-f92c-47eb-bc65-ef241ec91a00.png)

cookie是一小段存储在浏览器的文本，数据大小不超过4kb，发送数据的时候，cookie会在请求头里一并发送给服务器端，session对象是储存在服务器端，主要是用来存储用户会话的数据，SessionID需要存储在浏览器端，通常存储在cookie中，浏览器发送接口请求的时候，需要带着这个sessionID，服务器端就可以根据这个sessionID找出当前请求的用户是谁了

## Cookie，session storage和local storage的区别

html5中，web storage分两种情况，session storage，local storage

**session storage**是用于本地储存，一个会话中的数据，这些数据只有在同一会话中才能访问，并当会话结束后，数据就会销毁，即当浏览器关闭是，数据就会消失。session storage仅仅是会话级别的存储，不是持久化的存储。注意session storage和session不是一个东西，要注意

**local storage**是用于持久化的本地存储，除非主动删除数据，否则永不过期

**cookie**存储在浏览器的一小段文本数据，第2-n次向浏览器发送请求的时候，cookie都会一并发过去，cookie不应该用作前端的数据缓存

总结：

cookie是与服务器进行交互，是http规范中的一部分，web storage中的session storage和local storage都是用来做本地数据缓存的机制



## JWT token![img](https://cdn.nlark.com/yuque/0/2022/webp/21986264/1650304666904-b0f376d5-5cb2-4847-acf9-de4fa4dc6cbd.webp)

###  什么是JWT?

JSON Web Token(JWT)是一种开放标准(RFC7519)，它定义了一种紧凑而独立的方式，用于在各方之间作为JSON对象安全地传输信息。此信息是可以验证和信任的，因为它是经过数字签名的。

来源JWT官网（https://jwt.io/introduction）

通俗的来说，jwt就是应用认证的一种解决方案，在讨论jwt之前，需要先了解的传统的session认证和token认证区别。

1.1 传统的session认证步骤：

客户端向服务端发送账号密码进行认证。

服务端在校验账号密码正确之后，将当前用户的基本信息保存到当前会话（session）中，并将sessionid返回给客户端。

客户端在拿到sessionid之后将其保存到cookie中，并在以后的每次请求中都携带该sessionid。

服务端根据客户端传过来的sessionid对当前用户进行认证，判断是否合法。

1.1.1 session认证存在的问题：

所有的session都在服务端进行保存，当用户量不断增大的时候服务器的开销也会随之增大。

在一个应用的服务器有多台的情况下，针对一个用户，就必须在每台服务器上都维护一个相同的session，或者引入Redis等中间件对session进行统一的管理。

[
](https://blog.csdn.net/qq_41286145/article/details/120726638)

1.2 token认证步骤：

客户端向服务端发送账号密码进行认证。

服务端在校验账号密码正确之后，生成token字符串返回给客户端。

客户端收到token之后进行保存，并在之后的每一次请求中将该token放到请求头中一并发送。

服务端在收到客户端传递的token之后，对token进行验证，判断是否合法。

使用token进行认证的话，服务端不需要保存任何会话信息或者用户信息，所有的信息都分散保存在客户端中，客户端每次请求的时候自己带上token即可。

### JWT的构成

jwt是一个很长的字符串，中间用“.”分割成三部分，实际的jwt如下所示：



```bash
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJsZWUiLCJpc3MiOiJsZWUiLCJpZCI6IjEyMzQ1NiIsInVzZXJOYW1lIjoiYWRtaW4iLCJleHAiOjE2MzQwMTU2MTgsImlhdCI6MTYzNDAwODQxOCwianRpIjoiZTY4ODAwNzUtODA5Mi00ZTM0LWEzZDctYmQ1YmQ4ZTA0YjBkIn0.MOPUmnWzbFijlM_vGESF6YdBDvSW3QtIh1qS8i_yuPc
```



jwt的三个部分如下：

Header（头部）

Payload（负载）

Signature（签名）

[
](https://blog.csdn.net/qq_41286145/article/details/120726638)

- **Header**

Header部分是一个json字符串，保存的是jwt的元数据，格式如下：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

alg：签名的加密方式

typ：表示这个令牌（token）的类型（type），JWT令牌统一写为JWT

最后将Header的json字符串进行Base64编码之后就得到了jwt的第一部分



- **Payload**

Payload部分同样是一个json字符串，其中保存的是有效信息，这些信息主要包含三个部分：

1. 标准中注册的声明
2. 公共的声明
3. 私有的声明



1.标准中注册的声明（JWT官方规定的字段）：

iss：jwt签发者

sub：jwt所面向的用户

aud：接收jwt的一方

exp：jwt的过期时间，这个过期时间必须要大于签发时间

nbf：定义在什么时间之前，该jwt都是不可用的

iat：jwt的签发时间

jti：jwt的唯一身份标识，主要用来作为一次性token，从而回避重放攻击



2.公共的声明 ：

公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息，但不建议添加敏感信息，因为该部分在客户端可解密。



3.私有的声明 ：

私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

Payload示例：

```json
{
  "sub":"lee",
  "iss":"lee",
  "id":"123456",
  "exp":"1634015618",
  "iat":"1634008418",
  "jti":"e6880075-8092-4e34-a3d7-bd5bd8e04b0d",
  "userName":"admin"
}
```

最后将Payload的json字符串进行Base64编码之后就得到了jwt的第二部分



- **Signature**

jwt的第三部分就是签名部分，针对以下三个部分进行加密即可：

**Header（Base64编码后的）**

**Payload（Base64编码后的）**

**secret**

将Base64编码后的Header和Payload用“.”进行连接，然后使用Header中的加密算法进行加盐secret加密。

```json
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

最后加密出的字符串就是jwt的第三部分。

JWT的使用

当客户端接收到服务端发来的jwt字符串之后，将其保存到Cookie或者localStorage里面，然后每次向服务器发送请求的时候都带上JWT，将jwt放到HTTP请求的头信息Authorization中。

```json
Authorization: Bearer <token>
```

3.1 服务端验证步骤

当服务端接收到客户端的token字符串之后，首先对字符串的第一部分进行解码，得到Header。

取出Header中的加密方式，使用该加密方式和secret对字符串的第一部分和第二部分进行加密，如果得到的签名和传递过来的签名一致，则认为该token合法。

对第二部分进行解码，得到需要的数据。

### 建议

1. Payload中不要存放敏感信息。
2. 保存好secret私钥，不要泄露。
3. jwt的有效期需要根据应用来决定，时间不宜设置过长。
4. 如果可以，请使用HTTPS。





REF：

b站老猿说开发

https://blog.csdn.net/qq_41286145/article/details/120726638

https://www.jianshu.com/p/29b92789b6a7