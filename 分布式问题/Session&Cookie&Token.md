Session、Cookie、Token
===

Cookie 和 Session
---

HTTP 协议是一种无状态协议，即每次服务端接收到客户端的请求时，都是一个全新的请求，服务器并不知道客户端的历史请求记录；Session 和 Cookie 的主要目的就是为了弥补 HTTP 的无状态特性。

Session 是什么
---

客户端请求服务端，服务端会为这次请求开辟一块**内存空间**，这个对象便是 Session 对象，存储结构为**ConcurrentHashMap**。Session 弥补了 HTTP 无状态特性，服务器可以利用 Session 存储客户端在同一个会话期间的一些操作记录。

Session 如何判断是否是同一会话
---

服务器第一次接收到请求时，开辟了一块 Session 空间（创建了Session对象），同时生成一个 sessionId ，并通过响应头的 **Set-Cookie：JSESSIONID=XXXXXXX**命令，向客户端发送要求设置 Cookie 的响应； 客户端收到响应后，在本机客户端设置了一个 **JSESSIONID=XXXXXXX**的 Cookie 信息，该 Cookie 的过期时间为浏览器会话结束；

接下来客户端每次向同一个网站发送请求时，请求头都会带上该 Cookie信息（包含 sessionId ）， 然后，服务器通过读取请求头中的 Cookie 信息，获取名称为 JSESSIONID 的值，得到此次请求的 sessionId。

Session 的缺点
---

Session 机制有个缺点，比如 A 服务器存储了 Session，就是做了负载均衡后，假如一段时间内 A 的访问量激增，会转发到 B 进行访问，但是 B 服务器并没有存储 A 的 Session，会导致 Session 的失效。

Cookies 是什么
---

HTTP 协议中的 Cookie 包括 Web Cookie 和浏览器 Cookie，它是服务器发送到 Web 浏览器的一小块数据。服务器发送到浏览器的 Cookie，浏览器会进行存储，并与下一个请求一起发送到服务器。通常，它用于判断两个请求是否来自于同一个浏览器，例如用户保持登录状态。

Cookie 主要用于下面三个目的

_会话管理_
登陆、购物车、游戏得分或者服务器应该记住的其他内容

_个性化_
用户偏好、主题或者其他设置

_追踪记录和分析用户行为_
Cookie 曾经用于一般的客户端存储。虽然这是合法的，因为它们是在客户端上存储数据的唯一方法，但如今建议使用现代存储 API。Cookie 随每个请求一起发送，因此它们可能会降低性能（尤其是对于移动数据连接而言）。

创建 Cookie
---

当接收到客户端发出的 HTTP 请求时，服务器可以发送带有响应的 Set-Cookie 标头，Cookie 通常由浏览器存储，然后将 Cookie 与 HTTP 标头一同向服务器发出请求。

Set-Cookie 和 Cookie 标头
----

Set-Cookie HTTP 响应标头将 cookie 从服务器发送到用户代理。

有两种类型的 Cookies，一种是 **Session Cookies**，一种是 **Persistent Cookies**，
如果 Cookie **不包含到期日期**，则将其视为会话 Cookie。会话 Cookie 存储在内存中，永远不会写入磁盘，当浏览器关闭时，此后 Cookie 将永久丢失。
如果 Cookie 包含**有效期** ，则将其视为持久性 Cookie。在到期指定的日期，Cookie 将从磁盘中删除。

还有一种是 Cookie的 Secure 和 HttpOnly 标记，下面依次来介绍一下

Cookie 的 Secure 和 HttpOnly 标记
----

安全的 Cookie 需要经过 HTTPS 协议通过加密的方式发送到服务器。即使是安全的，也不应该将敏感信息存储在cookie 中，因为它们本质上是不安全的，并且此标志不能提供真正的的保护。

HttpOnly 的作用
-----

会话 Cookie 中缺少 HttpOnly 属性会导致攻击者可以通过程序(JS脚本、Applet等)获取到用户的 Cookie  信息，造成用户 Cookie 信息泄露，增加攻击者的跨站脚本攻击威胁。

HttpOnly 是微软对 Cookie 做的扩展，该值指定 Cookie 是否可通过客户端脚本访问。

如果在 Cookie 中没有设置 HttpOnly 属性为 true，可能导致 Cookie 被窃取。窃取的 Cookie 可以包含标识站点用户的敏感信息，如 ASP.NET 会话 ID 或 Forms 身份验证票证，攻击者可以重播窃取的 Cookie，以便伪装成用户或获取敏感信息，进行跨站脚本攻击等。

Cookie 的作用域
----

Domain 和 Path 标识定义了 Cookie 的作用域：即 Cookie 应该发送给哪些 URL。
Domain 标识指定了哪些主机可以接受 Cookie。如果不指定，默认为当前主机(不包含子域名）。如果指定了Domain，则一般包含子域名。
例如，如果设置 Domain=mozilla.org，则 Cookie 也包含在子域名中（如developer.mozilla.org）。
例如，设置 Path=/docs，则以下地址都会匹配：

/docs
/docs/Web/
/docs/Web/HTTP

**JSON Web Token 和 Session Cookies 的对比**
---

JSON Web Token ，简称 JWT，它和 Session都可以为网站提供用户的身份认证，但是它们不是一回事。

下面是 JWT 和 Session 不同之处的研究

JWT 和 Session Cookies 的相同之处
----

在探讨 JWT 和 Session Cookies 之前，有必要需要先去理解一下它们的相同之处。
它们既可以对用户进行身份验证，也可以用来在用户单击进入不同页面时以及登陆网站或应用程序后进行身份验证。
如果没有这两者，那你可能需要在每个页面切换时都需要进行登录了。因为 HTTP 是一个无状态的协议。这也就意味着当你访问某个网页，然后单击同一站点上的另一个页面时，服务器的内存中将不会记住你之前的操作。

因此，如果你登录并访问了你有权访问的另一个页面，由于 HTTP 不会记录你刚刚登录的信息，因此你将再次登录。
JWT 和 Session Cookies 就是用来处理在不同页面之间切换，保存用户登录信息的机制。
也就是说，这两种技术都是用来保存你的登录状态，能够让你在浏览任意受密码保护的网站。通过在每次产生新的请求时对用户数据进行身份验证来解决此问题。
所以 JWT 和 Session Cookies 的相同之处是什么？那就是它们能够支持你在发送不同请求之间，记录并验证你的登录状态的一种机制。

什么是 Session Cookies
-----

Session Cookies 也称为会话 Cookies，在 Session Cookies 中，用户的登录状态会保存在服务器的内存中。当用户登录时，Session 就被服务端安全的创建。
在每次请求时，服务器都会从会话 Cookie 中读取 SessionId，如果服务端的数据和读取的 SessionId 相同，那么服务器就会发送响应给浏览器，允许用户登录。

什么是 Json Web Tokens
-----

Json Web Token 的简称就是 JWT，通常可以称为 Json 令牌。它是RFC 7519 中定义的用于安全的将信息作为 Json 对象进行传输的一种形式。JWT 中存储的信息是经过数字签名的，因此可以被信任和理解。可以使用 HMAC 算法或使用 RSA/ECDSA 的公用/专用密钥对 JWT 进行签名。
使用 JWT 主要用来下面两点

**认证(Authorization)**：这是使用 JWT 最常见的一种情况，一旦用户登录，后面每个请求都会包含 JWT，从而允许用户访问该令牌所允许的路由、服务和资源。单点登录是当今广泛使用 JWT 的一项功能，因为它的开销很小。

**信息交换(Information Exchange)**：JWT 是能够安全传输信息的一种方式。通过使用公钥/私钥对 JWT 进行签名认证。此外，由于签名是使用 head 和 payload 计算的，因此你还可以验证内容是否遭到篡改。
