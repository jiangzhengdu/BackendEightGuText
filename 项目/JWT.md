JWT
===

什么是 Json Web Tokens
---

Json Web Token 的简称就是 JWT，通常可以称为 Json 令牌。它是RFC 7519 中定义的用于安全的将信息作为 Json 对象进行传输的一种形式。JWT 中存储的信息是经过数字签名的，因此可以被信任和理解。可以使用 HMAC算法或使用 RSA/ECDSA 的公用/专用密钥对 JWT 进行签名。
使用 JWT 主要用来下面两点

**认证(Authorization)**：这是使用 JWT 最常见的一种情况，一旦用户登录，后面每个请求都会包含 JWT，从而允许用户访问该令牌所允许的路由、服务和资源。单点登录是当今广泛使用 JWT 的一项功能，因为它的开销很小。
**信息交换(Information Exchange**)：JWT 是能够安全传输信息的一种方式。通过使用公钥/私钥对 JWT 进行签名认证。此外，由于签名是使用 head 和 payload 计算的，因此你还可以验证内容是否遭到篡改。

JWT 的格式
---

JWT 主要由三部分组成，每个部分用 . 进行分割，各个部分分别是 Header Payload Signature

Header
----

Header 是 JWT 的标头，它通常由两部分组成：令牌的类型(即 JWT)和使用的 签名算法，例如 HMAC SHA256 或 RSA。

``` json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

指定类型和签名算法后，Json 块被 Base64Url 编码形成 JWT 的第一部分。

Payload
----

Token 的第二部分是 Payload，Payload 中包含一个声明。声明是有关实体（通常是用户）和其他数据的声明。共有三种类型的声明：registered, public 和 private 声明。

**registered** 声明： 包含一组建议使用的预定义声明，主要包括

``` json
{
    "iss": "John Wu JWT",
    "iat": 1441593502,
    "exp": 1441594722,
    "aud": "www.example.com",
    "sub": "jrocket@example.com",
    "from_user": "B",
    "target_user": "A"
}
```

* iss (issuer) 签发人
* exp (expiration time) 过期时间
* sub (subject) 主题 该JWT所面向的用户
* aud (audience)受众 接收该JWT的一方
* nbf (Not Before)生效时间
* iat (Issued At)签发时间
* jti (JWT ID)编号

**public 声明**：公共的声明，可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息，但不建议添加敏感信息，因为该部分在客户端可解密。

**private 声明**：自定义声明，旨在在同意使用它们的各方之间共享信息，既不是注册声明也不是公共声明。

``` json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

然后 payload Json 块会被**Base64Url** 编码形成 JWT 的第二部分。

signature
---

JWT 的第三部分是一个签证信息，这个签证信息由三部分组成

1. header (base64后的)
2. payload (base64后的)
3. secret
比如我们需要 HMAC SHA256 算法进行签名

``` json
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名用于验证消息在此过程中没有更改，并且对于使用私钥进行签名的令牌，它还可以验证 JWT 的发送者的真实身份
现在我们把上面的三个由点分隔的 Base64-URL 字符串部分组成在一起，这个字符串可以在 HTML 和 HTTP 环境中轻松传递这些字符串。
最后一步签名的过程，实际上是对头部以及载荷内容进行签名。一般而言，加密算法对于不同的输入产生的输出总是不一样的。对于两个不同的输入，产生同样的输出的概率极其地小（有可能比我成世界首富的概率还小）。所以，我们就把“不一样的输入产生不一样的输出”当做必然事件来看待吧。

所以，如果有人对头部以及载荷的内容解码之后进行修改，再进行编码的话，那么新的头部和载荷的签名和之前的签名就将是不一样的。而且，如果不知道服务器加密的时候用的密钥的话，得出来的签名也一定会是不一样的。

服务器应用在接受到JWT后，会首先对头部和载荷的内容用同一算法再次签名。那么服务器应用是怎么知道我们用的是哪一种算法呢？别忘了，我们在JWT的头部中已经用alg字段指明了我们的加密算法了。

如果服务器应用对头部和载荷再次以同样方法签名之后发现，自己计算出来的签名和接受到的签名**不一样**，那么就说明这个Token的内容被别人动过的，我们应该拒绝这个Token，返回一个HTTP 401 Unauthorized响应。

JWT 和 Session Cookies 的不同
---

JWT 和 Session Cookies 都提供安全的用户身份验证，但是它们有以下几点不同
**密码签名**
JWT 具有加密签名，而 Session Cookies 则没有。

**JSON 是无状态的**
JWT 是无状态的，因为声明被存储在客户端，而不是服务端内存中。
身份验证可以在本地进行，而不是在请求必须通过服务器数据库或类似位置中进行。 这意味着可以对用户进行多次身份验证，而无需与站点或应用程序的数据库进行通信，也无需在此过程中消耗大量资源。

**可扩展性**
Session Cookies 是存储在服务器内存中，这就意味着如果网站或者应用很大的情况下会耗费大量的资源。由于 JWT 是无状态的，在许多情况下，它们可以节省服务器资源。因此 JWT 要比 Session Cookies 具有更强的可扩展性。
**JWT 支持跨域认证**
Session Cookies 只能用在单个节点的域或者它的子域中有效。如果它们尝试通过第三个节点访问，就会被禁止。如果你希望自己的网站和其他站点建立安全连接时，这是一个问题。
使用 JWT 可以解决这个问题，使用 JWT 能够通过多个节点进行用户认证，也就是我们常说的跨域认证。
**JWT 和 Session Cookies 的选型**
我们上面探讨了 JWT 和 Cookies 的不同点，相信你也会对选型有了更深的认识，大致来说
对于只需要登录用户并访问存储在站点数据库中的一些信息的中小型网站来说，Session Cookies 通常就能满足。
如果你有企业级站点，应用程序或附近的站点，并且需要处理大量的请求，尤其是第三方或很多第三方（包括位于不同域的API），则 JWT 显然更适合。

禁用 Cookies，如何使用 Session
---

如果禁用了 Cookies，服务器仍会将 sessionId 以 cookie 的方式发送给浏览器，但是，浏览器不再保存这个cookie (即sessionId) 了。

如果想要继续使用 session，需要采用 URL 重写 的方式来实现，在每个url后面自动加上PHPSESSID的值即可，用户禁止cookie后，服务器仍会将sessionId以cookie的方式发送给浏览器

什么场景该适合使用jwt？
---

来聊聊几个场景，注意，以下的几个场景不是都和jwt贴合。

一次性验证
----

比如用户注册后需要发一封邮件让其激活账户，通常邮件中需要有一个链接，这个链接需要具备以下的特性：能够标识用户，该链接具有时效性（通常只允许几小时之内激活），不能被篡改以激活其他可能的账户…这种场景就和 jwt 的特性非常贴近，jwt 的 payload 中固定的参数：iss 签发者和 exp 过期时间正是为其做准备的。

restful api的无状态认证
----

使用 jwt 来做 restful api 的身份认证也是值得推崇的一种使用方案。客户端和服务端共享 secret；过期时间由服务端校验，客户端定时刷新；签名信息不可被修改…spring security oauth jwt 提供了一套完整的 jwt 认证体系，以笔者的经验来看：使用 oauth2 或 jwt 来做 restful api 的认证都没有大问题，oauth2 功能更多，支持的场景更丰富，后者实现简单。

使用 jwt 做单点登录+会话管理(不推荐)
----

在《八幅漫画理解使用JSON Web Token设计单点登录系统》一文中提及了使用 jwt 来完成单点登录，本文接下来的内容主要就是围绕这一点来进行讨论。如果你正在考虑使用 jwt+cookie 代替 session+cookie ，我强力不推荐你这么做。

首先明确一点：使用 jwt 来设计单点登录系统是一个不太严谨的说法。首先 cookie+jwt 的方案前提是非跨域的单点登录(cookie 无法被自动携带至其他域名)，其次单点登录系统包含了很多技术细节，至少包含了身份认证和会话管理，这还不涉及到权限管理。如果觉得比较抽象，不妨用传统的 session+cookie 单点登录方案来做类比，通常我们可以选择 spring security（身份认证和权限管理的安全框架）和 spring session（session 共享）来构建，而选择用
jwt 设计单点登录系统需要**解决很多传统方案中同样存在和本不存在的问题**，以下一一详细罗列。

1. jwt token泄露了怎么办？

前面的文章下有不少人留言提到这个问题，我则认为这不是问题。传统的 session+cookie 方案，如果泄露了 sessionId，别人同样可以盗用你的身份。扬汤止沸不如釜底抽薪，不妨来追根溯源一下，什么场景会导致你的 jwt 泄露。

遵循如下的实践可以尽可能保护你的 jwt 不被泄露：使用 https 加密你的应用，返回 jwt 给客户端时设置 httpOnly=true 并且使用 cookie 而不是 LocalStorage 存储 jwt，这样可以防止 XSS 攻击和 CSRF 攻击（对这两种攻击感兴趣的童鞋可以看下 spring security 中对他们的介绍CSRF,XSS）
2. secret如何设计

jwt 唯一存储在服务端的只有一个 secret，个人认为这个 secret 应该设计成和用户相关的，而不是一个所有用户公用的统一值。这样可以有效的避免一些注销和修改密码时遇到的窘境。
3. 注销和修改密码

传统的 session+cookie 方案用户点击注销，服务端**清空 session** 即可，因为状态保存在服务端。但 jwt 的方案就比较难办了，因为 jwt 是**无状态的，服务端通过计算来校验有效性**。没有存储起来，所以即使客户端删除了 jwt，但是该 jwt 还是在有效期内，只不过处于一个游离状态。分析下痛点：注销变得复杂的原因在于 jwt 的无状态。我提供几个方案，视具体的业务来决定能不能接受。

* 仅仅清空客户端的 cookie，这样用户访问时就不会携带 jwt，服务端就认为用户需要重新登录。这是一个典型的假注销，对于用户表现出退出的行为，实际上这个时候携带对应的 jwt 依旧可以访问系统。

* 清空或修改服务端的用户对应的 secret，这样在用户注销后，jwt 本身不变，但是由于 secret 不存在或改变，则无法完成校验。这也是为什么将 secret 设计成和用户相关的原因。

* 借助第三方存储自己管理 jwt 的状态，可以以 jwt 为 key，实现去 redis 一类的缓存中间件中去校验存在性。方案设计并不难，但是引入 redis 之后，就把无状态的 jwt 硬生生变成了有状态了，违背了 jwt 的初衷。实际上这个方案和 session 都差不多了。

修改密码则略微有些不同，假设号被到了，修改密码（是用户密码，不是 jwt 的 secret）之后，盗号者在原 jwt 有效期之内依旧可以继续访问系统，所以仅仅**清空 cookie 自然是不够的，这时，需要强制性的修改 secret**。在我的实践中就是这样做的。
4. 续签问题

续签问题可以说是我抵制使用 jwt 来代替传统 session 的最大原因，因为 jwt 的设计中我就没有发现它将续签认为是自身的一个特性。传统的 cookie 续签方案一般都是框架自带的，session 有效期 30 分钟，30 分钟内如果有访问，session 有效期被刷新至 30 分钟。而 jwt 本身的 payload 之中也有一个 exp 过期时间参数，来代表一个 jwt 的时效性，而 jwt 想延期这个 exp 就有点身不由己了，因为 payload 是参与签名的，一旦过期时间被修改，整个Jwt 串就变了，jwt 的特性天然不支持续签！

如果你一定要使用 jwt 做会话管理（payload 中存储会话信息），也不是没有解决方案，但个人认为都不是很令人满意

* 每次请求刷新 jwt
jwt 修改 payload 中的 exp 后整个 jwt 串就会发生改变，那…就让它变好了，每次请求都返回一个新的 jwt 给客户端。太暴力了，不用我赘述这样做是多么的不优雅，以及带来的性能问题。
但，至少这是最简单的解决方案。
* 只要快要过期的时候刷新 jwt

一个上述方案的改造点是，只在最后的几分钟返回给客户端一个新的 jwt。这样做，触发刷新 jwt 基本就要看运气了，如果用户恰巧在最后几分钟访问了服务器，触发了刷新，万事大吉；如果用户连续操作了 27 分钟，只有最后的 3 分钟没有操作，导致未刷新 jwt，无疑会令用户抓狂。

* 完善 refreshToken

借鉴 oauth2 的设计，返回给客户端一个 refreshToken，允许客户端主动刷新 jwt。一般而言，jwt 的过期时间可以设置为数小时，而 refreshToken 的过期时间设置为数天。
我认为该方案并可行性是存在的，但是为了解决 jwt 的续签把整个流程改变了，为什么不考虑下 oauth2 的 password 模式和 client 模式呢？

* 使用 redis 记录独立的过期时间

实际上我的项目中由于历史遗留问题，就是使用 jwt 来做登录和会话管理的，为了解决续签问题，我们在 redis 中单独会每个 jwt 设置了过期时间，每次访问时刷新 jwt 的过期时间，若 jwt 不存在与 redis 中则认为过期。

tips:精确控制 redis 的过期时间不是件容易的事

同样改变了 jwt 的流程，不过嘛，世间安得两全法。我只能奉劝各位还未使用 jwt 做会话管理的朋友，尽量还是选用传统的 session+cookie 方案，有很多成熟的分布式 session 框架和安全框架供你开箱即用。

这么长一个字符串，还不如我把数据存到数据库，给一个长的很难碰撞的key来映射，也就是专用token。

这位兄弟认为 jwt 太长了，是不是可以考虑使用和 oauth2 一样的 uuid 来映射。这里面自然是有问题的，jwt 不仅仅是作为身份的认证（验证签名是否正确，签发者是否存在，有限期是否过期），还在其 **payload 中存储着会话信息**，这是 jwt 和 session 的最大区别，一个在客户端携带会话信息，一个在服务端存储会话信息。如果真的是要将 jwt 的信息置于在共享存储中，那再找不到任何使用 jwt 的意义了。
jwt 和 oauth2 都可以用于 restful 的认证，就我个人的使用经验来看，spring security oauth2 可以很好的使用多种认证模式：client 模式，password 模式，implicit 模式（authorization code 模式不算单纯的接口认证模式），也可以很方便的实现权限控制，什么样的 api 需要什么样的权限，什么样的资源需要什么样的 scope…而 jwt 我只用它来实现过身份认证，功能较单一
