JWT
===

什么是 Json Web Tokens
---

Json Web Token 的简称就是 JWT，通常可以称为 Json 令牌。它是RFC 7519 中定义的用于安全的将信息作为 Json 对象进行传输的一种形式。JWT 中存储的信息是经过数字签名的，因此可以被信任和理解。可以使用 HMAC 算法或使用 RSA/ECDSA 的公用/专用密钥对 JWT 进行签名。
使用 JWT 主要用来下面两点

**认证(Authorization)**：这是使用 JWT 最常见的一种情况，一旦用户登录，后面每个请求都会包含 JWT，从而允许用户访问该令牌所允许的路由、服务和资源。单点登录是当今广泛使用 JWT 的一项功能，因为它的开销很小。
**信息交换(Information Exchange**)：JWT 是能够安全传输信息的一种方式。通过使用公钥/私钥对 JWT 进行签名认证。此外，由于签名是使用 head 和 payload 计算的，因此你还可以验证内容是否遭到篡改。

JWT 的格式
---

下面，我们会探讨一下 JWT 的组成和格式是什么

JWT 主要由三部分组成，每个部分用 . 进行分割，各个部分分别是

Header
Payload
Signature

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

iss (issuer)
签发人

exp (expiration time)
过期时间

sub (subject)
主题

aud (audience)
受众

nbf (Not Before)
生效时间

iat (Issued At)
签发时间

jti (JWT ID)
编号

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

header (base64后的)
payload (base64后的)
secret

比如我们需要 HMAC SHA256 算法进行签名

``` json
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名用于验证消息在此过程中没有更改，并且对于使用私钥进行签名的令牌，它还可以验证 JWT 的发送者的真实身份

拼凑在一起
现在我们把上面的三个由点分隔的 Base64-URL 字符串部分组成在一起，这个字符串可以在 HTML 和 HTTP 环境中轻松传递这些字符串。
下面是一个完整的 JWT 示例，它对 header 和 payload 进行编码，然后使用 signature 进行签名

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
