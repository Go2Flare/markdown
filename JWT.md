[jwt官网](https://jwt.io/#debugger-io)

# JSON Web Token 入门教程

作者： [阮一峰](https://www.ruanyifeng.com/)

日期： [2018年7月23日](https://www.ruanyifeng.com/blog/2018/07/)

JSON Web Token（缩写 JWT）是目前最流行的跨域认证解决方案，本文介绍它的原理和用法。

![img](https://www.wangbase.com/blogimg/asset/201807/bg2018072301.jpg)

## 一、跨域认证的问题

互联网服务离不开用户认证。一般流程是下面这样。

> 1、用户向服务器发送用户名和密码。
>
> 2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
>
> 3、服务器向用户返回一个 session_id，写入用户的 Cookie。
>
> 4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
>
> 5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

这种模式的问题在于，扩展性（scaling）不好。单机当然没有问题，如果是服务器集群，或者是跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。

举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

## 二、JWT 的原理

JWT 的原理是，服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样。

> ```javascript
> {
>   "姓名": "张三",
>   "角色": "管理员",
>   "到期时间": "2018年7月1日0点0分"
> }
> ```

以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名（详见后文）。

服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。

## 三、JWT 的数据结构

实际的 JWT 大概就像下面这样。

![img](https://www.wangbase.com/blogimg/asset/201807/bg2018072304.jpg)

它是一个很长的字符串，中间用点（`.`）分隔成三个部分。注意，JWT 内部是没有换行的，这里只是为了便于展示，将它写成了几行。

JWT 的三个部分依次如下。

> - Header（头部）
> - Payload（负载）
> - Signature（签名）

写成一行，就是下面的样子。

> ```javascript
> Header.Payload.Signature
> ```

![img](https://www.wangbase.com/blogimg/asset/201807/bg2018072303.jpg)

下面依次介绍这三个部分。

### 3.1 Header

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子。

> ```javascript
> {
>   "alg": "HS256",
>   "typ": "JWT"
> }
> ```

上面代码中，`alg`属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；`typ`属性表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`。

最后，将上面的 JSON 对象使用 Base64URL 算法（详见后文）转成字符串。

### 3.2 Payload

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。

> - iss (issuer)：签发人
> - exp (expiration time)：过期时间
> - sub (subject)：主题
> - aud (audience)：受众
> - nbf (Not Before)：生效时间
> - iat (Issued At)：签发时间
> - jti (JWT ID)：编号

除了官方字段，你还可以在这个部分定义私有字段，下面就是一个例子。

> ```javascript
> {
>   "sub": "1234567890",
>   "name": "John Doe",
>   "admin": true
> }
> ```

注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。

这个 JSON 对象也要使用 Base64URL 算法转成字符串。

### 3.3 Signature

Signature 部分是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

> ```javascript
> HMACSHA256(
>   base64UrlEncode(header) + "." +
>   base64UrlEncode(payload),
>   secret)
> ```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。

### 3.4 Base64URL

前面提到，Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符`+`、`/`和`=`，在 URL 里面有特殊含义，所以要被替换掉：`=`被省略、`+`替换成`-`，`/`替换成`_` 。这就是 Base64URL 算法。

## 四、JWT 的使用方式

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息`Authorization`字段里面。

> ```javascript
> Authorization: Bearer <token>
> ```

另一种做法是，跨域的时候，JWT 就放在 POST 请求的数据体里面。

## 五、JWT 的几个特点

（1）JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。

（2）JWT 不加密的情况下，不能将秘密数据写入 JWT。

（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。

（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

（6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

## 六、参考链接

- [Introduction to JSON Web Tokens](https://jwt.io/introduction/)， by Auth0
- [Sessionless Authentication using JWTs (with Node + Express + Passport JS)](https://medium.com/@bryanmanuele/sessionless-authentication-withe-jwts-with-node-express-passport-js-69b059e4b22c), by Bryan Manuele
- [Learn how to use JSON Web Tokens](https://github.com/dwyl/learn-json-web-tokens/blob/master/README.md), by dwyl

（完）

### 文档信息

- 版权声明：自由转载-非商用-非衍生-保持署名（[创意共享3.0许可证](https://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)）
- 发表日期： 2018年7月23日



## 相关文章

- 2022.01.28: [命令行常用工具的替代品](https://www.ruanyifeng.com/blog/2022/01/cli-alternative-tools.html)

  程序员离不开命令行，许多经典命令是每天必用的，比如ls和cd。

- 2021.09.07: [《C 语言入门教程》发布了](https://www.ruanyifeng.com/blog/2021/09/c-language-tutorial.html)

  向大家报告，我写了一本《C 语言入门教程》，已经上线了，欢迎访问。

- 2021.08.26: [最适合程序员的笔记软件](https://www.ruanyifeng.com/blog/2021/08/best-note-taking-software-for-programmers.html)

  程序员的笔记软件，应该满足下面几个条件。

- 2021.05.10: [软件工程的最大难题](https://www.ruanyifeng.com/blog/2021/05/scaling-problem.html)

  一、引言 大学有一门课程《软件工程》，研究如何组织和管理软件项目。

## 留言（160条）

***\*小胡子哥\** 说：**

可以说讲的很清楚了

2018年7月23日 17:58 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391309) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*moyo\** 说：**

没质量，这种网上一抓一大把。

2018年7月23日 18:28 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391311) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*jiajunhuang\** 说：**

无状态的东西好在方便扩展，例如map reduce。不过通常服务器还是需要对session或者token有比较高的掌控权。JWT还是用在一些不那么需要保证安全的地方会好一些例如确认退订邮件等。

2018年7月23日 18:29 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391312) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*anix\** 说：**

看懂了，哈哈，算是入门了吗？

2018年7月23日 18:41 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391313) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*anton\** 说：**

清晰简洁，????

2018年7月23日 18:51 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391314) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*令狐洋葱\** 说：**

如今安全越来越重要，客户端储存永久token，是否有它适用的场景？

2018年7月23日 18:54 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391315) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Krist Pan\** 说：**

最近的博客质量不高啊，不过高质量的博文不常有，为了保证文章数量，只能这样了。

2018年7月23日 19:22 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391316) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Simon\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

这都能酸..

2018年7月23日 19:36 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391317) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*seven\** 说：**

> ```
> 引用令狐洋葱的发言：
> ```
>
> 如今安全越来越重要，客户端储存永久token，是否有它适用的场景？

并没建议让永久储存token啊. 看特点的第5条

2018年7月23日 21:54 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391320) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*优惠券\** 说：**

（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了

以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

2018年7月24日 08:42 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391323) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*山西大学\** 说：**

对于行外人来说，喜欢了解这样的原理性的东西。实话讲，我虽然看不懂，但是希望了解框架知识就够了。

2018年7月24日 09:21 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391336) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*皮皮\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

大哥啊，别喷了，如果你懂，那么请你默默的离开，简单明了本身就是质量。

2018年7月24日 10:04 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391340) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*hooyes\** 说：**

感觉缺点大于优点。

2018年7月24日 10:19 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391343) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Orange\** 说：**

完全不懂的我竟然看懂了 谁敢说没质量！！！

2018年7月24日 10:33 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391344) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*ivan\** 说：**

> ```
> 引用优惠券的发言：
> ```
>
> （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
> 也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了
>
> 以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

JWT最大特点不就是状态存储在客户端么，可以实现多点登录，服务端不用做很多的额外工作

2018年7月24日 10:59 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391346) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*microjan\** 说：**

很好,看懂了.

2018年7月24日 15:17 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391350) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*swift\** 说：**

session和jwt都有各自的使用场景，并不是一概而论好坏，各自都有优缺点，取长补短就行了

2018年7月24日 15:58 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391354) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*anonym\** 说：**

> ```
> 引用优惠券的发言：
> ```
>
> （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
> 也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了
>
> 以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

不引入数据库支持的话这个的确没法实现。最好的方案是维护一张基于 jti (JWT ID) 的表，将jti和user_id关联起来，生成新的jwt时将该user_id旧的jwt标记为失效就好，具体场景还得看你们的具体业务逻辑。

2018年7月24日 18:48 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391358) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*CCTV\** 说：**

jwt主要还是用在移动端的

2018年7月24日 19:01 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391359) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*mm\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

有一个老乞丐，天天到捷运站磕头乞讨，小白每次经过，固定都会给他二十元，每天下班遇到就给，这样持续了一年。

有一天，小白只给乞丐十元，乞丐愣了一下，但还是勉强抬头笑了笑。

隔天，小白又给十元。再隔天，又是十元。

到了第七天，老乞丐终于忍不住了，伸手拉住小白说：「以前你都给二十，怎么现在都只给十元？」

小白被抓住，有点吃惊，旋即回过神来说：「啊，抱歉，我结婚了，要省着点。」

乞丐怒得一巴掌打过去：「什么玩意]

2018年7月25日 10:08 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391376) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*优惠券\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

软一峰绝对不是编程界代码写的最厉害的，但他一定是编程界教程写的最好的

2018年7月25日 11:56 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391380) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*一位过客\** 说：**

写的好,简单明了

2018年7月25日 13:28 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391382) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*stong\** 说：**

通俗易懂~ 小白有个疑问望各位大神解答：
用jwt来实现 跨域单点登录：
①A系统登录成功—>jwt 返回tokemn ->客户端保存至 cookie
②同一浏览器进入B 系统-> 浏览器提交token ->服务器验证token 是否有效 ->有效B系统自动登陆用户

问：1） jwt 不易保存私密信息，A登陆后 必然将用户部分信息存储至 token B才能获取是哪位用户？
2） B验证token 验证登陆后 不在b系统中产生sesion ?

2018年7月25日 14:15 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391385) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*WalkingSun\** 说：**

> ```
> 引用优惠券的发言：
> ```
>
> （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
> 也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了
>
> 以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

签名可以加时间戳或这其他随机参与加密，一旦登录就更新签名，之前的签名失效，不会出现多个在线啦

2018年7月25日 14:27 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391386) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*小白\** 说：**

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage

浏览器存在 cookie 中的token 跨域也是非共享的， A系统登录后，浏览器访问B 怎么拿到这个token?

2018年7月25日 14:45 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391388) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Aecced\** 说：**

JWT的几个特点中第四条，除非服务器部署额外的逻辑，能不能举例说明下，有点困惑

2018年7月25日 15:18 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391392) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*阮老师的小粉丝\** 说：**

@优惠券：

你可以让他对比维护是否签发token的清单和维护session列表哪个更容易一些，如果在这方面JWT没有优势，也就没有重新发明的必要额。

2018年7月25日 17:59 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391400) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*ixx\** 说：**

> ```
> 引用优惠券的发言：
> ```
>
> （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
> 也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了
>
> 以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

JWT 解决的最大问题是跨域，如果你们的业务不涉及跨域完全没有必要用JWT session 就够用了，如果有跨域需要，那么改造Session实现跨域访问，要比改造JWT实现单用户登陆复杂的多

2018年7月26日 10:41 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391417) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*ixx\** 说：**

> ```
> 引用stong的发言：
> ```
>
> 通俗易懂~ 小白有个疑问望各位大神解答：
> 用jwt来实现 跨域单点登录：
> ①A系统登录成功—>jwt 返回tokemn ->客户端保存至 cookie
> ②同一浏览器进入B 系统-> 浏览器提交token ->服务器验证token 是否有效 ->有效B系统自动登陆用户
>
> 问：1） jwt不易保存私密信息，A登陆后 必然将用户部分信息存储至 token B才能获取是哪位用户？
> 2）B验证token 验证登陆后 不在b系统中产生sesion ?

使用JWT用户数据由客户端传入，每个请求都会带着这些信息，也就不需要后端生成Session了，在每次请求的时候验证登陆状态及登陆用户

2018年7月26日 10:45 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391418) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*zw\** 说：**

阮老师，不聊聊yimiao时间吗

2018年7月26日 10:49 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391419) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*ed\** 说：**

等了好久阮老师终于也写jwt了，之前网上查的确实没阮老师写的易懂。

2018年7月26日 11:31 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391422) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*donotlb\** 说：**

> ```
> 引用小白的发言：
> ```
>
> 客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage
>
> 浏览器存在 cookie 中的token 跨域也是非共享的， A系统登录后，浏览器访问B怎么拿到这个token?

跨域这块，前端也是要处理的，两种情况：

\1. token 存储在 cookie 里，按传统的跨域方案，共享/传递 cookie 里的 token
\2. token 存储在 localStorage 里，则需要 localStorage 的跨域方案，以便 共享/传递 cookie 里的 token，localStorage 的跨域方案可以搜搜，有开源类库

2018年7月26日 16:22 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391435) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*王嘉\** 说：**

周五了，催更了，哈哈哈

2018年7月27日 10:52 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391472) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Value\** 说：**

@优惠券：

如果是移动端的话，可以考虑签发的时候加入手机的设备编号作为验证。

2018年7月30日 15:48 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391617) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*缺木\** 说：**

JWT是否只是提供一种思路模式，并没有实际的什么依赖需要添加到系统中，文章中写的官方提供的7个字段以及签名算法是否其实并没有实际的效果，其实只是类似一种规范而已，而开发者自己根据规范去实现？

2018年7月31日 16:26 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391658) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*animateJoke\** 说：**

能否讲解一下，JWT跨二级域名记录登录状态时怎么实现同步登录和退出，之前项目token是存在sessionStorage里面的，但是为了跨二级域名登录，没找到实现的方法就放到了cookie里面，感觉有点不太好，有没有不放到cookie的解决办法

2018年8月 2日 17:00 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391729) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*创建人\** 说：**

博客写得很好，很喜欢

2018年8月 8日 19:56 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-391955) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*RichardWang\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

是一抓一大把 但是写的调理这么清楚的只有一家。我之前也做过jwt token的项目，去看英文的文章很多篇才弄清楚的。

2018年8月10日 14:56 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392019) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*清启\** 说：**

> ```
> 引用令狐洋葱的发言：
> ```
>
> 如今安全越来越重要，客户端储存永久token，是否有它适用的场景？

没让你永久储存啊，何况你永久储存那也是你的事情，但是啥时候过期是服务端实现啊，你拿着过期的token请求api，会返回错误

2018年8月10日 16:50 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392022) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*清启\** 说：**

@优惠券：

使用jwt的场景是前后端彻底分离，无法通过session来认证的。默认的确是可以多次登录，发了多个token，也能改成只发一个啊，让前面的失效，这和session是2回事情。拒绝使用jwt那么授权接口怎么知道是哪个用户在请求？

2018年8月10日 16:55 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392023) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*清启\** 说：**

> ```
> 引用CCTV的发言：
> ```
>
> jwt主要还是用在移动端的

还可以用于前后端分离，对后端来说，没有移动端不移动端的概念，反正你们都是调用接口，我管你是pc，h5 or app

2018年8月10日 16:57 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392024) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*清启\** 说：**

@stong：

1.其实jwt里面有加密的uid，或者是uid的对应关系

2.b系统产生不产生session，是你b系统的验证登录逻辑决定啊，你要从session验证用户，那就要写session，你如果是从redis验证token，就不需要写session

2018年8月10日 17:01 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392025) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*清启\** 说：**

@小白：

可以去参考CAS的方案。或者我简单说一下ucenter的方案，a系统登录后，拿到了jwt，访问一下b系统的一个链接，由b系统的此链接将jwt存入localStorage

2018年8月10日 17:04 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392027) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*清启\** 说：**

> ```
> 引用Aecced的发言：
> ```
>
> JWT的几个特点中第四条，除非服务器部署额外的逻辑，能不能举例说明下，有点困惑

第四条是JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。？

其实第四条说的有点绝对了，原文第四条的意思应该是jwt颁发后，一般扩展包没提供让其失效方法。但是要让jwt失效依然很简单，因为jwt一般会放在redis或者mysql表，只要逻辑上去找到uid对应的jwt，删了就可以了。

2018年8月10日 17:08 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392029) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*清启\** 说：**

> ```
> 引用ixx的发言：
> ```
>
> 
>
> JWT 解决的最大问题是跨域，如果你们的业务不涉及跨域完全没有必要用JWTsession 就够用了，如果有跨域需要，那么改造Session实现跨域访问，要比改造JWT实现单用户登陆复杂的多

个人认为jwt解决最大的问题不是跨域，而是前后端分离后，纯接口方面的用户认证问题。

2018年8月10日 17:11 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392030) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*清启\** 说：**

> ```
> 引用缺木的发言：
> ```
>
> JWT是否只是提供一种思路模式，并没有实际的什么依赖需要添加到系统中，文章中写的官方提供的7个字段以及签名算法是否其实并没有实际的效果，其实只是类似一种规范而已，而开发者自己根据规范去实现？

如你所说，jwt是一种规范，你也可以不按那个字段走，但是不用自己实现，你可以去全球最大的交友网站github上找到你要用，一般别人写的都按规范走了，而且方法都已封装好。

2018年8月10日 17:14 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392031) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*joe\** 说：**

jwt 验证怎么才能够防止重放攻击呢? 最近正在重构自己的个人网站,想做的更安全一点,有人说用挑战应答?有人说用timeStamp + nonce? 有哪位同仁能解答一下吗?

2018年8月16日 01:57 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392223) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*algery\** 说：**

@清启：

```
但是要让jwt失效依然很简单，因为jwt一般会放在redis或者mysql表，只要逻辑上去找到uid对应的jwt，删了就可以了。
```

你这个不就是额外的逻辑了

2018年8月19日 10:03 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392310) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*泫\** 说：**

@mm：

哈哈，说的真好！话说阮老师有没有捐赠地址啥的。看了挺多好文章了。

2018年8月20日 11:46 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392365) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*ahuigo\** 说：**

@优惠券：

\1. session+jwt 结合
\2. session 只存：uid:lastTime, 别什么数据都往session 丢。用lastTime确定是否过期
\3. 密码修改、A/B登录，lastTime都要更新。
\4. 类似腾讯这种，同时支持pc+mobile，就用：uid:{pc:lastTime, mobile:lastTime}

2018年8月21日 01:25 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392387) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*黄豆\** 说：**

很简单明了，看了就懂，赞

2018年8月23日 09:28 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392448) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Jerry\** 说：**

通俗易懂 ~ 可以说说 refresh token 吗？

2018年8月23日 21:32 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392469) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Tiger\** 说：**

那是不是一旦黑客截取到JWT就意味着在有效期内就可以为所欲为了？

2018年8月27日 20:03 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392612) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*BobDu\** 说：**

我觉得只有在安全要求一般，且需要跨域的情况下才可以考虑JWT技术。

毕竟JWT签发后确实是服务器无法控制，如果你非得要通过维护一张JWT id表单来控制JWT失效，那和session比没有任何优势，反而还引入了安全性问题，得不偿失。

至于说到集群共享session的问题，通过分布式redis应该是可以很好的解决这一问题。

2018年9月 3日 20:00 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392793) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*AberSheeran\** 说：**

@BobDu：

JWT签发之后为什么要用JWT id表？JWT里面带失效时间不就行了？只要私钥不泄露又没法伪造。

2018年9月 4日 21:40 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392832) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*curry\** 说：**

> ```
> 引用WalkingSun的发言：
> ```
>
> 
>
> 签名可以加时间戳或这其他随机参与加密，一旦登录就更新签名，之前的签名失效，不会出现多个在线啦

难道每个用户一个secret ？

2018年9月 5日 15:01 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392843) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*blue water\** 说：**

哪位大佬能讲一下怎么把token放入HTTP 请求的头信息Authorization字段里面........求解（最好有代码）

2018年9月 6日 13:47 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392879) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*hejg\** 说：**

阮老师的博客能推送微信公众号就更好了，能更方便更及时的看到。。

2018年9月 7日 17:08 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392940) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*泫\** 说：**

> ```
> 引用hejg的发言：
> ```
>
> 阮老师的博客能推送微信公众号就更好了，能更方便更及时的看到。。

RSS订阅了解一下

2018年9月 9日 11:13 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-392973) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*云有你\** 说：**

阮老师的语文水平和我的一样高，因为我能读懂。

2018年9月27日 17:23 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-393465) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*云泥\** 说：**

大神，清晰、简洁，要是能要实际应用例子就更好了，手动点赞

2018年9月30日 11:26 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-393565) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*李鲁杨\** 说：**

> ```
> 引用小白的发言：
> ```
>
> 客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage
>
> 浏览器存在 cookie 中的token 跨域也是非共享的， A系统登录后，浏览器访问B怎么拿到这个token?

如果你要访问B的时候携带 Token 过去，那把 Token 放在 Header参数，Url的Path参数，Query参数，Body参数都可以传过去，不一定非要放在 cookie 里面啊， Token 一般还是放在请求头里面的

2018年10月16日 10:31 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-394071) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*luojing\** 说：**

@优惠券：

oauth2 了解一下

2018年10月18日 10:04 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-394169) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*freyxian\** 说：**

@优惠券：

应用服务器端保留用户的最新token即可。

2018年10月25日 09:01 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-394366) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*aws\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

有质量的人会能说出来你这样的话？
JWT本来就这点东西，怎么算有质量?

2018年10月26日 21:33 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-394450) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*diamond\** 说：**

博主介绍了如何生成token，却没有介绍server端收到token之后如何验证啊

2018年10月29日 19:43 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-394523) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*黄大虾\** 说：**

> ```
> 引用diamond的发言：
> ```
>
> 博主介绍了如何生成token，却没有介绍server端收到token之后如何验证啊

我也想问这个问题

2018年11月29日 09:29 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-396177) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*狗饼\** 说：**

> ```
> 引用WalkingSun的发言：
> ```
>
> 
>
> 签名可以加时间戳或这其他随机参与加密，一旦登录就更新签名，之前的签名失效，不会出现多个在线啦

那不是服务器还要保存签名?

2018年12月 7日 16:01 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-396742) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*周华飞\** 说：**

通俗易懂。站在你的肩膀上，我可以看的更远。感谢！

2018年12月24日 17:32 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-398541) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*zard\** 说：**

这是翻译官方的吧

2018年12月27日 07:36 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-398908) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*JS&SJ\** 说：**

> ```
> 引用Simon的发言：
> ```
>
> 
>
> 这都能酸..

不愿看就悄悄走开，没有开源分享精神还跑出来指点江山。。。

2018年12月27日 19:43 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-399022) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*JS&SJ\** 说：**

> ```
> 引用Simon的发言：
> ```
>
> 
> 这都能酸..


Sorry,刚来引用错了，我是支持您怼那个嘴欠moyo的“没质量，这种网上一抓一大把。”的傻评论

2018年12月27日 19:49 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-399024) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*wangx\** 说：**

请问payload是什么作用，为什么出现2遍（payload、signature）。
只有加密的不行吗？

2019年1月 7日 11:12 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-401541) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*darron\** 说：**

“举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。”

请教一下，session数据持久化怎么能解决“A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录”这个问题呢？A网站和B网站cookie不共享啊。

2019年1月10日 22:37 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-402218) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Bob\** 说：**

简单明了，谢谢。

2019年1月15日 10:40 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-403826) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*hengsha\** 说：**

后端用redis或数据库记录一个用户id到token的对应表，token不用有任何意义，也不需要加密解密，随机字符串即可。

客户端发送token的方法跟JWT一样，可以选择用cookie、header或url参数，服务器校验根据 seesion -> 用户id 的公共存储判断。这样可以很简单地在后台根据用户最后访问时间更新token有效时间和自动删除过期token ，还能通过这个表查看在线用户，这些都是JWT不具备的功能。

所以我看不出JWT相比这个方案有什么好处，除了能节约服务器上 token 到 用户ID 表的这一次查询，还有其他好处吗？ 而且一般业务请求，除了拿到用户ID，还有更多需要服务器调用的服务，比如拿到用户完整对象、用户权限、用户业务操作对象等等，这些不比 查询 token 到用户ID表的开销大得多吗？

请大伙指正。

2019年1月16日 14:24 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-404404) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*genericyzh\** 说：**

> ```
> 引用优惠券的发言：
> ```
>
> （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
> 也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了
>
> 以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

无法反驳

2019年1月16日 19:48 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-404514) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*韩版女装\** 说：**

> ```
> 引用优惠券的发言：
> ```
>
> （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
> 也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了
>
> 以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

在数据库里或服务端存储机构保存用户等陆状态，如果登录状态发生变化，则重新签发一个包含登录状态码的JWT给客户端，原来的A设备因为登录状态码对不上号而被退出，A设备退出后，如果再次登录时再次重新签一个包含登录状态码的JWT给客户端，这样A手机重新登录后，B手机因为登录状态码对不上而被退出。

注意下：每次使用登录功能登录时登录时，客户端向服务端提交登录数据时，会连同发送重新设定登录状态码的指令，登录状态码为随机字符串。

这是我的想法，哈俣，大神不要喷，毕竟我是新的人。

2019年2月11日 19:24 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-408127) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*让蜡烛代替所有灯\** 说：**

什么时候能讲讲jhipster相关的东西呢阮老师

2019年2月15日 10:53 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-408234) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*梁大大\** 说：**

阮大神些的教程是我见过最易懂的。没有之一。赞一个！！！

2019年2月22日 10:09 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-408340) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*廖权名\** 说：**

@hengsha：

数据库挂了呢，用户量大需要的空间又是多大。

2019年2月22日 15:54 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-408380) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*赵庆华\** 说：**

JWT的作用是什么? 他就是用来在两个独立系统间建立信任而不需要服务端之间进行通讯. 建立信任之后,你的服务器该做什么session还是要做的.
A为业务系统
B为授信系统

认证过程
A打开B 传入A的标识进行登录授权，B根据A的标识使用对应TOKEN生成JWT
A获取到JWT，进行校验，匹配的情况下，A生成SESSION返回浏览器前端
退出登录过程
A收到退出登录请求，或者关闭浏览器，session失效，下次登录重复此流程

2019年3月 6日 09:34 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-409739) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*甘训奏\** 说：**

> ```
> 引用优惠券的发言：
> ```
>
> （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
> 也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了
>
> 以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

防止重复登陆，都是通过其他策略，例如：互斥ip控制，互斥设备控制等。token更多的是认证，而不是防重复。

2019年3月 8日 11:11 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-409786) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*韩忠康\** 说：**

"举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。"

个人感觉这种解决方案不是解决单点登录问题。单点登录问题的核心是如何让浏览器拥有目标网站的会话标识。

2019年3月29日 09:37 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-410213) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*kevingui\** 说：**

阮老师的文章从来都是这么通俗易懂，中文教程中最实用的入门内容

2019年4月 8日 10:31 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-410413) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*游学者\** 说：**

@hengsha：

你的策略很好，我喜欢。

但是，试想，如果在你这套机制之上，将无意义的token换成JWT， 在一些需要显示用户昵称、头像的场景（不敏感场景），减少一次数据库查询，又何尝不可呢？ （对于需要使用JWT里面的敏感数据如用户ID时，你就查一下数据库呗。）

那你可能会提出，那我就将我那一串无意义的token换成有意义的昵称、头像，不就好了吗。对于这个问题，你要试想，你没有签名，黑客就能伪造昵称、头像甚至是用户ID，然后提交到服务器上，服务器上也会接收这个昵称、头像甚至是用户ID，那就麻烦大了。

那你可能会提出，那我也搞一个签名，防止黑客伪造不就行了？那你不觉得，你这就是JWT吗，当然你可能不是JSON格式的，那就是AWT、BWT、CWT、XWT，随便啦，其实最终就都是一样的东西了。

2019年4月13日 11:22 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-410550) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*哈哈哈\** 说：**

JWT跟跨域有个啥的关系。。。。。。。。

2019年4月16日 17:25 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-410585) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*zbh\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

网上肯定有类似的东西，但是阮老师讲的言简意赅，且易懂

2019年4月24日 14:53 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-410790) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*someone\** 说：**

槽点太多，JWT和跨域有什么关系？？BASE64是加密算法？TOKEN放到url里是因为要满足URL规范所以要urlencode 可是这和BASE64有什么关系？

2019年5月 7日 17:21 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-411046) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*wocacaca\** 说：**

有个问题 假如A网站登入后B网站要自动登录，前端把数据存在哪里 不管是cookie 还是 localstorage 如果A和B不在同一个域名下应该是没有办法获取到的吧，比如我把相关token存在A.com的localstorage里，在进入B.com 的时候 localstorage里是没有token的呀？？？？？？

2019年5月 9日 14:38 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-411072) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*哈哈哈\** 说：**

> ```
> 引用ixx的发言：
> ```
>
> JWT 解决的最大问题是跨域，如果你们的业务不涉及跨域完全没有必要用JWTsession 就够用了，如果有跨域需要，那么改造Session实现跨域访问，要比改造JWT实现单用户登陆复杂的多

扯鸡巴蛋，改造Session怎么就复杂了。服务端不用动，客户端自己添加Cookie就完事了。在Node作为前端服务器，Servlet做后台服务端的情况下。Node自己解析SetCookie:响应头，然后发送第二次访问时，携带Cookie:JESSIONID=xxxx请求头过来不就行了吗。

2019年5月28日 15:01 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-411423) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*大大大\** 说：**

防止JWT被篡改关键就在最后的签名：当客户端拿着JWT来服务器读取数据时，服务器会对Header和Payload以及秘钥再进行一次签名，和客户端返回的签名进行比对，如果不一致，就是被篡改了。

Header以及Payload都是Base64URL的编码，对应着就可以解码（任何人都可以，因为没有涉及到加密方式）。因此，重要信息不要放在这里面，比如同时存放用户名和密码。

好处我觉得很大啊：第一个，就是服务器的内存中不用和session机制那样存储用户状态数据了，分不式的我还没学到，不乱说；第二个，也不用把token存储在数据库了，如果存数据库，每次去数据库查询开销都不小（redis里存的话，和session区别就不大了，都是占用内存）。

另外，突然想到了如何让账号只能在一台设备登录？我看了各种文章都是登录的时候，在Redis里来保存JWT，客户端每次请求接口判断本次携带的JWT是否和Redis里保存的一直，不一致则登录失效。那如果这样做，感觉JWT的优势就没有了，有没有其他方法可以解决JWT同时只能有一个账号在线的问题？

2019年5月29日 17:43 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-411444) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*husese\** 说：**

这个是json web token 的翻译吧

2019年7月10日 10:09 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-412129) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*欧阳彦祖\** 说：**

大佬，这篇文我转载了，转载到我的公众号，会标记署名和原文链接的，如果有不便，可以发邮件联系我，我这边马上删除，大家也可以关注一下我的公众号哦，公众号名称：请快点喜欢我。

2019年7月15日 16:41 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-412213) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*熊~\** 说：**

是如何保证同一个用户每次生成的token是不一样的呢？

2019年7月24日 20:37 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-412376) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Hon\** 说：**

没有有【有状态】 token 和【无状态】 token 的对比，和相关的应用场景。

2019年7月28日 16:47 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-412459) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*jackletter\** 说：**

jwt的最大有点应该是应用于分布式系统和减轻服务器存储状态信息的开销
不过缺点也很明显：
\1. 服务器无法控制token失效
\2. token到期失效后的自动刷新
解决方案：
一 token的到期后失效的问题可以使后台服务器针对每次请求都刷新过期时间（即使有是服务器集群也没关系，拿到刷新后的token在前端直接覆盖就行）。
二 服务器控制token失效这个问题必须在后台记录已经颁发的token，但一般这种操作是允许有延迟的并且频率不是太高，所以并发的问题不是太严重。
详细的解决方案会在我的博客中介绍（正准备写。。。）https://blog.csdn.net/u010476739/article/details/98757471

2019年8月 7日 16:25 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-412617) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*tmyyss\** 说：**

其实JWT是可以控制token的失效的

2019年8月18日 10:27 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-412799) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*tmyyss\** 说：**

可以通过控制密钥的方式来实现token的失效

2019年8月18日 14:06 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-412800) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*小星星\** 说：**

关于jwt 过期的问题。很多人说 可以在服务端记录 点什么东西来实现。
都这样做了。那还叫jwt吗，， 就和 原来的session 本质上没区别了。
jwt的核心就是 服务端不存东西。
传统的session。自己实现的话，那一样可以 服务端只存 sessionid+过期时间
然后数据加密给客户端。。

2019年8月20日 11:42 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-412845) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Arcry\** 说：**

A访问服务器时返回了token，B拿此token访问是不是能获取A的权限呢？

2019年8月28日 17:49 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-413037) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*zicjin\** 说：**

> ```
> 引用韩版女装的发言：
> ```
>
> 
>
> 在数据库里或服务端存储机构保存用户等陆状态，如果登录状态发生变化，则重新签发一个包含登录状态码的JWT给客户端，原来的A设备因为登录状态码对不上号而被退出，A设备退出后，如果再次登录时再次重新签一个包含登录状态码的JWT给客户端，这样A手机重新登录后，B手机因为登录状态码对不上而被退出。
>
> 注意下：每次使用登录功能登录时登录时，客户端向服务端提交登录数据时，会连同发送重新设定登录状态码的指令，登录状态码为随机字符串。
>
> 这是我的想法，哈俣，大神不要喷，毕竟我是新的人。

完全没有理解JWT的意义，你这样表示所有请求全部都要去检查某个单点数据库里的所谓“登陆状态码”，那等于就是一个session。没有哪个数据库吃得消，必须用分布式内存存储。

2019年8月30日 14:24 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-413102) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*胡庆杰\** 说：**

cookie是可以跨域的，CSRF正是利用了这一点
这说明浏览器自动收发cookie是有利有弊的

2019年9月15日 12:02 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-413484) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*mecode\** 说：**

有人能解答下下面????这三个问题吗
受众（audience） 具体指的是什么（最好举例说明）？
主题（subject）具体指的是什么（最好举例说明）？
subject 与 audience 有什么区别？

2019年10月12日 15:58 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-413954) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*六月和饭\** 说：**

写得不错，比我刚看的另一篇文章好理解些

2019年10月12日 16:44 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-413956) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*celine\** 说：**

> ```
> 引用WalkingSun的发言：
> ```
>
> 
>
> 签名可以加时间戳或这其他随机参与加密，一旦登录就更新签名，之前的签名失效，不会出现多个在线啦

如果签名里加入了时间戳或者其他随机参数，那服务器端还得持久化每个用户的签名密钥，也就失去了jwt后端不需要存东西的优点了吧。

2019年11月 6日 15:04 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-414380) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*ZHZ\** 说：**

如何做到每次请求都会自动延长过期时间。

2019年11月22日 12:33 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-414711) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*zsran\** 说：**

简单明了 竟然看懂了

2019年11月27日 11:49 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-414784) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*kantappa\** 说：**

高手啊，写得通俗易懂

2019年12月16日 14:21 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-415163) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Hsing\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

尽管一大把，看了几篇，但是我认为这篇是相对质量高的一篇

2019年12月20日 22:16 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-415319) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*wztscau\** 说：**

> ```
> 引用hooyes的发言：
> ```
>
> 感觉缺点大于优点。

我也觉得，始终客户端保存登录信息不可靠，也不够强大。还是老的session有用

2020年1月 6日 17:55 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-415708) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*wzts\** 说：**

还有你不会的吗？

2020年1月 7日 14:05 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-415715) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Qingz\** 说：**

不错，比网上大部分的都详细很多。
一看就懂，不废话

2020年1月18日 16:19 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-415964) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*KSM\** 说：**

感觉你把“相同账号异地登录相斥”和“多点登录混淆”？

请再次查看使用JWT的场景：A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录；

你的场景：“做到B登录后让A过期”，不适合用JWT吧？

2020年1月21日 11:36 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-416005) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Trinyoung\** 说：**

如果照阮老师这么写，通过jwt除了加密，就和cookie没有任何区别了。如果把用户数据转成token，保存在服务端，每次请求携带token，如果用户信息数据比较大，那么将严重占用网络带宽。另外把用户数据放在客户端，那么根本就防止不了csrf 攻击。

2020年4月 9日 17:31 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-417601) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*ct29\** 说：**

> ```
> 引用优惠券的发言：
> ```
>
> （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
> 也就是说一个用户在手机A中登录了，然后又在手机B中登录，在过期之前手机A和B都可以登录，无法做到B登录后让A过期，如果要做到这点，就必须让服务器维护一个清单（记录该账号是否已经签发token），这样又回到session的老路了
>
> 以上就是我们公司后端拒绝使用jwt的理由，我该怎么驳倒他？

可以在payload中记录id，在签发token的时候后端记录id和token的map，验证token的时候判断是否后端有记录即可

2020年4月13日 15:24 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-417702) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*大不点\** 说：**

非常好的博文，已经入门，跟我公司内部说的jwt加密一毛一样呀

2020年4月13日 17:22 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-417704) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*儒斌\** 说：**

HMACSHA256(
base64UrlEncode(header) + "." +
base64UrlEncode(payload),
secret)

为什么还把秘钥生成签名放在token里， 服务端已经保存了secret，为什么还把他传到前端增加被暴露的风险？

2020年4月16日 23:02 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-417741) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*deping\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

关键就是阮一峰的博客就是质量的保证。别的地方我未必相信正确性。再说有个集中的地方不好吗？本来就是这些内容，你有期待说些什么？怎么不见你对互联网知识做这样的贡献？

2020年4月18日 18:10 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-417820) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*hec9527\** 说：**

> ```
> 引用ct29的发言：
> ```
>
> 可以在payload中记录id，在签发token的时候后端记录id和token的map，验证token的时候判断是否后端有记录即可

可以在payload中多增加几个属性，用于对用户进行分类，然后服务器只需要维护一个当前使用的分类列表， 不在列表中的403， 优点就是可以初步筛选token，缺点就是没办法删除指定用户的token

2020年4月21日 10:24 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-417872) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Bob Linus\** 说：**

3.3节的过程不应该叫签名吧，只是用了消息验证码。这里客户端没有密钥，不用解密，解密还是下次发给服务器时进行的。也就是说，自己的消息验证码由自己验证，目的只是防止JWT传给客户端后被篡改。理解对不对？

2020年4月29日 09:11 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-418059) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*vipcxj\** 说：**

一开始有点理解错了，以为jwt是客户端生成的，后来想想不合理。现在的理解如下，客户端用传统方式登陆一个服务端，用户名密码手机什么的都行，然后服务端用自己不公开密钥签名一个带用户身份信息的jwt字符串发给客户端，之后客户端自然就能用这个jwt令牌来向服务端表明自己的身份。至于如何实现单点登录，多个服务器应该会共享密钥，这样就能验证令牌真伪，然后各个服务端可以用jwt中的用户身份信息来判断是否可以登陆自己的服务器，这样就实现了单点登录。因为每个服务端共用一个密钥，只要约定相同格式的jwt，一个jwt令牌自然就能各个服务端通用了。对于限制重复登陆，一个方法是将客户端ip写在jwt里，这样ip不对不让登陆。难题应该在于不一定能准确获得用户ip，不过短时间内，就算是假的ip，也不会老是变吧，至少不会出现登陆一会又不能登陆的情况。

2020年5月31日 21:11 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-418823) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*vipcxj\** 说：**

> ```
> 引用儒斌的发言：
> ```
>
> HMACSHA256(
> base64UrlEncode(header) + "." +
> base64UrlEncode(payload),
> secret)
>
> 为什么还把秘钥生成签名放在token里， 服务端已经保存了secret，为什么还把他传到前端增加被暴露的风险？

并不是放在token里，而是作为生成签名的参数，生成后的签名里面是找不到密钥的影子的。加密领域递归调用是很常见的。

2020年5月31日 21:13 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-418824) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*ganoatri\** 说：**

> ```
> 引用moyo的发言：
> ```
>
> 没质量，这种网上一抓一大把。

光会喷的人网上也是一抓一大把。觉得没质量不如你推荐个更好的？

2020年6月10日 08:25 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-419117) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*麦昆\** 说：**

为什么要把服务端使用的加密方式alg声明在JWT的header中呢？虽然密钥在服务端，但是也同样增加了暴露和破解的风险呀

2020年7月 1日 14:21 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-419916) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*你好世界\** 说：**

最后的总结不错，看懂了。

2020年7月18日 15:07 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-420471) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*龚欣\** 说：**

阮老师，token放在cookie里面，除了不能跨域，还有不能抵抗csrf攻击吧？

2020年10月18日 16:29 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-423308) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*龚欣\** 说：**

> ```
> 引用Trinyoung的发言：
> ```
>
> 如果照阮老师这么写，通过jwt除了加密，就和cookie没有任何区别了。如果把用户数据转成token，保存在服务端，每次请求携带token，如果用户信息数据比较大，那么将严重占用网络带宽。另外把用户数据放在客户端，那么根本就防止不了csrf 攻击。

这个问题我也没想明白，token如果不放在cookie，是不是就可以抵抗csrf。加上jwt有加密过，所以即使存在客户端，别人也不能轻易获取到。不知道我理解的对不对？

2020年10月18日 16:34 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-423309) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*海门\** 说：**

有时候你觉得简单，是因为别人写的清晰明了，而不是自己多厉害。

2020年11月12日 12:59 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-423737) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*在线等\** 说：**

jwt中的jti的用法给讲讲呗，尤其是防重放攻击，如何做到的？

2020年12月 8日 18:04 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-424205) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Petty Fox\** 说：**

说真的这篇文章至少从客观层面简明、准确的阐述了JWT协议以及特点。
写的很棒。

2020年12月18日 12:40 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-424378) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*Petty Fox\** 说：**

> ```
> 引用麦昆的发言：
> ```
>
> 为什么要把服务端使用的加密方式alg声明在JWT的header中呢？虽然密钥在服务端，但是也同样增加了暴露和破解的风险呀

1，如果使用的签名算法本身已经有风险，那么不管你告不告诉攻击者它都能破解，因此透露算法就好似再说，签名协议在这里，有本事来破解。我们所认知的签名算法不都是公开的吗，所以破解的难点不在于是否知道是哪个签名算法。

2，在协议头声明算法策略，能保证签名算法的升级、共享、兼容的特性。

3，真正坚固的算法策略，是暴露该暴露的，仅守护自己坚决不能泄露的“密钥”。就如加密算法RSA，透明的加密流程，满世界跑的公钥，但就是不可泄露私钥。

2020年12月18日 12:54 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-424379) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*myhlcb\** 说：**

阮老师的文章牛逼的地方就是用最简练的语言把要说明的对象表达的很清楚
网上很多文章或许更详细，但是伴随着的是裹脚布一样的废话

2020年12月21日 18:17 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-424433) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*AlbertS\** 说：**

> ```
> 引用tmyyss的发言：
> ```
>
> 其实JWT是可以控制token的失效的

请问这个怎么处理，还有token刷新要怎么实现呢？

2021年1月 7日 16:06 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-424841) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*小白白\** 说：**

谢谢博主的文章，支持！感谢分享知识！

2021年2月18日 09:59 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-425500) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*磨刀匠\** 说：**

JWT token最大的问题，不在于安全，而是在于token不能携带太多内容，否则每次请求包太大影响性能，内容少就必然导致token验证通过后，还是需要根据token内容查缓存的用户相关信息，这就回到了session分布式缓存一样的问题，采用token跨域到另外的服务器后，仍然绕不过去查询登录用户缓存数据问题

2021年3月 5日 16:32 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-425782) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*多肉动物\** 说：**

@优惠券：

老兄，我也是苦恼这个问题，你后来成功干翻了他没有？

2021年3月18日 09:48 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-425990) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*张小牛\** 说：**

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，就可以返回给用户。

这里需要说明一下，算出的Signature也要经过Base64URL处理，处理后再和其他两个字符串拼接，才是对的。

2021年3月19日 09:57 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-426013) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*江无花\** 说：**

第三遍刷这篇文章了，看到了很多以为没有关注点，阮老师的文章就是耐看

2021年3月24日 01:38 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-426069) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*士大夫给\** 说：**

入门说没质量？？？

2021年3月27日 19:48 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-426135) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*请\** 说：**

有啥是峰子不会的?

2021年4月 9日 15:50 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-426310) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*PETE\** 说：**

我不太懂。既然每次请求都会传送token，而且token中包括了secret。如果被劫取，header，payload又都是明文加密，为什么会安全呢。

2021年4月13日 05:09 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-426352) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*aaaa\** 说：**

> ```
> 引用PETE的发言：
> ```
>
> 我不太懂。既然每次请求都会传送token，而且token中包括了secret。如果被劫取，header，payload又都是明文加密，为什么会安全呢。

secret是生成签名时所使用的盐值，对于生成的结果，你是不可能反推出secret的

2021年4月17日 13:43 | [#](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-426431) | [引用](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#comment-text)

***\*maooar\** 说：**

JWT不应该保存到后端，缓存，数据库，都不行！
怎么在后端让JWT失效？评论里有说在生成JWT时候保存到redis,mysql...,然后每次请求从redis，mysql查找，找不到就算失效。这个想法有问题，拿着无状态的JWT却用session的思路实现验证逻辑，不觉得搞笑吗？
无状态就是后端不保存任何JWT信息，只负责签发和验证。脱离了这一条就不叫无状态，应该用session类似机制更合适。

**一.CSRF是什么？**

　　CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。

**二.CSRF可以做什么？**

　　你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全。

**三.CSRF漏洞现状**

　　CSRF这种攻击方式在2000年已经被国外的安全人员提出，但在国内，直到06年才开始被关注，08年，国内外的多个大型社区和交互网站分别爆出CSRF漏洞，如：NYTimes.com（纽约时报）、Metafilter（一个大型的BLOG网站），YouTube和百度HI......而现在，互联网上的许多站点仍对此毫无防备，以至于安全业界称CSRF为“沉睡的巨人”。

**四.CSRF的原理**

　　下图简单阐述了CSRF攻击的思想：

　　![img](https://pic002.cnblogs.com/img/hyddd/200904/2009040916453171.jpg)

　　从上图可以看出，要完成一次CSRF攻击，受害者必须依次完成两个步骤：

　　1.登录受信任网站A，并在本地生成Cookie。

　　2.在不登出A的情况下，访问危险网站B。

　　看到这里，你也许会说：“如果我不满足以上两个条件中的一个，我就不会受到CSRF的攻击”。是的，确实如此，但你不能保证以下情况不会发生：

　　1.你不能保证你登录了一个网站后，不再打开一个tab页面并访问另外的网站。

　　2.你不能保证你关闭浏览器了后，你本地的Cookie立刻过期，你上次的会话已经结束。（事实上，关闭浏览器不能结束一个会话，但大多数人都会错误的认为关闭浏览器就等于退出登录/结束会话了......）

　　3.上图中所谓的攻击网站，可能是一个存在其他漏洞的可信任的经常被人访问的网站。

 

　　上面大概地讲了一下CSRF攻击的思想，下面我将用几个例子详细说说具体的CSRF攻击，这里我以一个银行转账的操作作为例子（仅仅是例子，真实的银行网站没这么傻:>）

　　**示例1：**

　　银行网站A，它以GET请求来完成银行转账的操作，如：http://www.mybank.com/Transfer.php?toBankId=11&money=1000

　　危险网站B，它里面有一段HTML的代码如下：

　　<img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>

　　首先，你登录了银行网站A，然后访问危险网站B，噢，这时你会发现你的银行账户少了1000块......

　　为什么会这样呢？原因是银行网站A违反了HTTP规范，使用GET请求更新资源。在访问危险网站B的之前，你已经登录了银行网站A，而B中的<img>以GET的方式请求第三方资源（这里的第三方就是指银行网站了，原本这是一个合法的请求，但这里被不法分子利用了），所以你的浏览器会带上你的银行网站A的Cookie发出Get请求，去获取资源“http://www.mybank.com/Transfer.php?toBankId=11&money=1000”，结果银行网站服务器收到请求后，认为这是一个更新资源操作（转账操作），所以就立刻进行转账操作......

　　**示例2：**

　　为了杜绝上面的问题，银行决定改用POST请求完成转账操作。

　　银行网站A的WEB表单如下：　　

　　<form action="Transfer.php" method="POST">　　　　<p>ToBankId: <input type="text" name="toBankId" /></p>　　　　<p>Money: <input type="text" name="money" /></p>　　　　<p><input type="submit" value="Transfer" /></p>　　</form>

　　后台处理页面Transfer.php如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　<?php
　　　　session_start();
　　　　if (isset($_REQUEST['toBankId'] &&　isset($_REQUEST['money']))
　　　　{
　　　　  buy_stocks($_REQUEST['toBankId'],　$_REQUEST['money']);
　　　　}
　　?>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　危险网站B，仍然只是包含那句HTML代码：

　　<img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>

　　和示例1中的操作一样，你首先登录了银行网站A，然后访问危险网站B，结果.....和示例1一样，你再次没了1000块～T_T，这次事故的原因是：银行后台使用了$_REQUEST去获取请求的数据，而$_REQUEST既可以获取GET请求的数据，也可以获取POST请求的数据，这就造成了在后台处理程序无法区分这到底是GET请求的数据还是POST请求的数据。在PHP中，可以使用$_GET和$_POST分别获取GET请求和POST请求的数据。在JAVA中，用于获取请求数据request一样存在不能区分GET请求数据和POST数据的问题。

　　**示例3：**

　　经过前面2个惨痛的教训，银行决定把获取请求数据的方法也改了，改用$_POST，只获取POST请求的数据，后台处理页面Transfer.php代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　<?php
　　　　session_start();
　　　　if (isset($_POST['toBankId'] &&　isset($_POST['money']))
　　　　{
　　　　  buy_stocks($_POST['toBankId'],　$_POST['money']);
　　　　}
　　?>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　然而，危险网站B与时俱进，它改了一下代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

<html>
　　<head>
<script type="text/javascript">
　　　　　　function steal()
　　　　　　{
     　　　　 iframe = document.frames["steal"];
　　   　   iframe.document.Submit("transfer");
　　　　　　}
　　　　</script>
　　</head>

　　<body onload="steal()">
　　　　<iframe name="steal" display="none">
　　　　　　<form method="POST" name="transfer"　action="http://www.myBank.com/Transfer.php">
　　　　　　　　<input type="hidden" name="toBankId" value="11">
　　　　　　　　<input type="hidden" name="money" value="1000">
　　　　　　</form>
　　　　</iframe>
　　</body>
</html>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果用户仍是继续上面的操作，很不幸，结果将会是再次不见1000块......因为这里危险网站B暗地里发送了POST请求到银行!

　　总结一下上面3个例子，CSRF主要的攻击模式基本上是以上的3种，其中以第1,2种最为严重，因为触发条件很简单，一个<img>就可以了，而第3种比较麻烦，需要使用JavaScript，所以使用的机会会比前面的少很多，但无论是哪种情况，只要触发了CSRF攻击，后果都有可能很严重。

　　理解上面的3种攻击模式，其实可以看出，CSRF攻击是源于WEB的隐式身份验证机制！WEB的身份验证机制虽然可以保证一个请求是来自于某个用户的浏览器，但却无法保证该请求是用户批准发送的！

**五.CSRF的防御**

　　我总结了一下看到的资料，CSRF的防御可以从服务端和客户端两方面着手，防御效果是从服务端着手效果比较好，现在一般的CSRF防御也都在服务端进行。

　　**1.服务端进行CSRF防御**

　　服务端的CSRF方式方法很多样，但总的思想都是一致的，就是在客户端页面增加伪随机数。

　　(1).Cookie Hashing(所有表单都包含同一个伪随机值)：

　　这可能是最简单的解决方案了，因为攻击者不能获得第三方的Cookie(理论上)，所以表单中的数据也就构造失败了:>

　　<?php
　　　　//构造加密的Cookie信息
　　　　$value = “DefenseSCRF”;
　　　　setcookie(”cookie”, $value, time()+3600);
　　?>

　　在表单里增加Hash值，以认证这确实是用户发送的请求。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　<?php
　　　　$hash = md5($_COOKIE['cookie']);
　　?>
　　<form method=”POST” action=”transfer.php”>
　　　　<input type=”text” name=”toBankId”>
　　　　<input type=”text” name=”money”>
　　　　<input type=”hidden” name=”hash” value=”<?=$hash;?>”>
　　　　<input type=”submit” name=”submit” value=”Submit”>
　　</form>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　然后在服务器端进行Hash值验证

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   <?php
　　   if(isset($_POST['check'])) {
   　   $hash = md5($_COOKIE['cookie']);
     　　 if($_POST['check'] == $hash) {
        　 doJob();
　　      } else {
　　　　　　　　//...
     　　 }
　　   } else {
　　　　　　//...
　　   }
   ?>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这个方法个人觉得已经可以杜绝99%的CSRF攻击了，那还有1%呢....由于用户的Cookie很容易由于网站的XSS漏洞而被盗取，这就另外的1%。一般的攻击者看到有需要算Hash值，基本都会放弃了，某些除外，所以如果需要100%的杜绝，这个不是最好的方法。
　　(2).验证码

　　这个方案的思路是：每次的用户提交都需要用户在表单中填写一个图片上的随机字符串，厄....这个方案可以完全解决CSRF，但个人觉得在易用性方面似乎不是太好，还有听闻是验证码图片的使用涉及了一个被称为MHTML的Bug，可能在某些版本的微软IE中受影响。

　　(3).One-Time Tokens(不同的表单包含一个不同的伪随机值)

　　在实现One-Time Tokens时，需要注意一点：就是“并行会话的兼容”。如果用户在一个站点上同时打开了两个不同的表单，CSRF保护措施不应该影响到他对任何表单的提交。考虑一下如果每次表单被装入时站点生成一个伪随机值来覆盖以前的伪随机值将会发生什么情况：用户只能成功地提交他最后打开的表单，因为所有其他的表单都含有非法的伪随机值。必须小心操作以确保CSRF保护措施不会影响选项卡式的浏览或者利用多个浏览器窗口浏览一个站点。

　　以下我的实现:

　　1).先是令牌生成函数(gen_token())：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   <?php
   function gen_token() {
　　　　//这里我是贪方便，实际上单使用Rand()得出的随机数作为令牌，也是不安全的。
　　　　//这个可以参考我写的Findbugs笔记中的[《Random object created and used only once》](https://www.cnblogs.com/hyddd/articles/1391737.html)
     $token = md5(uniqid(rand(), true));
     return $token;
   }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　2).然后是Session令牌生成函数(gen_stoken())：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   <?php
   　function gen_stoken() {
$pToken = "";
　　　　　　if($_SESSION[STOKEN_NAME] == $pToken){
　　　　　　　　//没有值，赋新值
　　　　　　　　$_SESSION[STOKEN_NAME] = gen_token();
　　　　　　}  
　　　　　　else{
　　　　　　　　//继续使用旧的值
　　　　　　}
   　}
   ?>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　3).WEB表单生成隐藏输入域的函数：　　

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   <?php
　　   function gen_input() {
   　   gen_stoken();
　　     echo “<input type=\”hidden\” name=\”" . FTOKEN_NAME . “\”
     　　   value=\”" . $_SESSION[STOKEN_NAME] . “\”> “;
   　　}
   ?>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　4).WEB表单结构：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   <?php
     session_start();
     include(”functions.php”);
   ?>
   <form method=”POST” action=”transfer.php”>
     <input type=”text” name=”toBankId”>
     <input type=”text” name=”money”>
     <? gen_input(); ?>
     <input type=”submit” name=”submit” value=”Submit”>
   </FORM>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　5).服务端核对令牌：

　　这个很简单，这里就不再啰嗦了。

　　上面这个其实不完全符合“并行会话的兼容”的规则，大家可以在此基础上修改。

 

　　其实还有很多想写，无奈精力有限，暂且打住，日后补充，如果错漏，请指出:>

　　PS：今天下午写这篇文档的时候FF崩溃了一次，写了一半文章的全没了，郁闷好久T_T.......

　　转载请说明出处，谢谢[hyddd(http://www.cnblogs.com/hyddd/)]