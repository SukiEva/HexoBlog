---
title: 应用层
toc: true
date: 2021-12-12 15:28
tags:
  - Network
categories: [Computer Basics,Computer Network]
updated: 2021-12-12 15:28
references:
  - title: LeetCode
    url: https://leetcode-cn.com/leetbook/read/networks-interview-highlights/eksi0s/
  - title: HTTP Headers - HTTP | MDN
    url: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers
---

**应用层**（**Application layer**）位于 [OSI模型](https://zh.wikipedia.org/wiki/OSI%E6%A8%A1%E5%9E%8B "OSI模型") 的第七层。应用层直接和应用程序接口结合，并提供常见的网络应用服务。

应用层的功能：

- 文件传输、访问和管理
- 电子邮件
- 虚拟终端
- 查询服务和远程作业登录

应用层的重要协议：

- HTTP（HyperText Transfer Protocol）超文本传输协议
- FTP（File Transfer Protocol）文件传输协议
	- TFTP（Trivial File Transfer Protocol）简单文件传输协议
- DNS（Domain Name System）域名系统
- SMTP（Simple Mail Transfer Protocol）简单邮件传输协议
	- POP3（Post Office Protocol - Version 3）邮局协议
- DHCP ( Dynamic Host Configuration Protocol）动态主机设置协议
- SNMP（Simple Network Management Protocol）简单网络管理协议

<!-- more -->

## HTTP

### HTTP 头部

HTTP 头字段根据实际用途被分为以下 4 种类型：

- 通用头字段 (General Header Fields)
- 请求头字段 (Request Header Fields)
- 响应头字段 (Response Header Fields)
- 实体头字段 (Entity Header Fields)

#### 通用头部

同时适用于请求和响应消息（客户端和服务器都可以使用），但与最终消息主体中传输的数据无关的消息头。

可以在客户端、服务器和其他应用程序之间提供一些非常有用的通用功能，如 Date 头部。

<table>
<thead>
<tr>
<th>协议头</th>
<th>说明</th>
<th>举例</th>
</tr>
</thead>
<tbody>
<tr>
<td>Cache-Control</td>
<td>用来指定当前的请求/回复中是否使用缓存机制</td>
<td>Cache-Control: no-store</td>
</tr>
<tr>
<td>Connection</td>
<td>客户端（浏览器）想要优先使用的连接类型</td>
<td>Connection: keep-alive (Upgrade)</td>
</tr>
<tr>
<td>Date</td>
<td>报文创建时间</td>
<td>Date: Dec, 26 Dec 2015 17: 30: 00 GMT</td>
</tr>
<tr>
<td>Trailer</td>
<td>会实现说明在报文主体后记录哪些首部字段，该首部字段可以使用在 HTTP/1.1 版本分块传输编码时</td>
<td>Trailer: Expiress</td>
</tr>
<tr>
<td>Transfer-Encoding</td>
<td>用来改变报文格式</td>
<td>Transfer-Encoding: chunked</td>
</tr>
<tr>
<td>Upgrade</td>
<td>要求服务器升级到一个高版本协议</td>
<td>Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11</td>
</tr>
<tr>
<td>Via</td>
<td>告诉服务器，这个请求是由哪些代理发出的</td>
<td>Via: 1.0 fred, 1.1 itbilu.com.com (Apache/1.1)</td>
</tr>
<tr>
<td>Warning</td>
<td>一个一般性的警告，表示在实体内容中可能存在错误</td>
<td>Warning: 199 Miscellaneous warning</td>
</tr>
</tbody>
</table>

#### 请求头部

包含更多有关要获取的资源或客户端本身信息的消息头，如 Accept 头部。

<table>
<thead>
<tr>
<th>协议头</th>
<th>说明</th>
<th>举例</th>
</tr>
</thead>
<tbody>
<tr>
<td>Accept</td>
<td>告诉服务器自己允许哪些媒体类型</td>
<td>Accept: text/plain</td>
</tr>
<tr>
<td>Accept-Charset</td>
<td>浏览器申明可接受的字符集</td>
<td>Accept-Charset: utf-8</td>
</tr>
<tr>
<td>Accept-Encoding</td>
<td>浏览器申明自己接收的编码方法</td>
<td>Accept-Encoding: gzip, deflate</td>
</tr>
<tr>
<td>Accept-Language</td>
<td>浏览器可接受的响应内容语言列表</td>
<td>Accept-Language: en-US</td>
</tr>
<tr>
<td>Authorization</td>
<td>用于表示 HTTP 协议中需要认证资源的认证信息</td>
<td>Authorization: Basic OSdjJGRpbjpvcGVul ANIc2SdDE==</td>
</tr>
<tr>
<td>Expect</td>
<td>表示客户端要求服务器做出特定的行为</td>
<td>Expect: 100-continue</td>
</tr>
<tr>
<td>From</td>
<td>发起此请求的用户的邮件地址</td>
<td>From: <a href="mailto:user@itbilu.com" target="_blank">user@itbilu.com</a></td>
</tr>
<tr>
<td>Host</td>
<td>表示服务器的域名以及服务器所监听的端口号</td>
<td>Host: <a href="http://www.itbilu.com:80" target="_blank"><www.itbilu.com:80</a></td>>
</tr>
<tr>
<td>If-XXX</td>
<td>条件请求</td>
<td>If-Modified-Since: Dec, 26 Dec 2015 17:30:00 GMT</td>
</tr>
<tr>
<td>Max-Forwards</td>
<td>限制该消息可被代理及网关转发的次数</td>
<td>Max-Forwards: 10</td>
</tr>
<tr>
<td>Range</td>
<td>表示请求某个实体的一部分，字节偏移以 0 开始</td>
<td>Range: bytes=500-999</td>
</tr>
<tr>
<td>Referer</td>
<td>表示浏览器所访问的前一个页面，可以认为是之前访问页面的链接将浏览器带到了当前页面</td>
<td>Referer: <a href="http://itbilu.com/nodejs" target="_blank"><http://itbilu.com/nodejs</a></td>>
</tr>
<tr>
<td>User-Agent</td>
<td>浏览器的身份标识字符串</td>
<td>User-Agent: Mozilla/……</td>
</tr>
</tbody>
</table>

#### 响应头部

包含有关响应的补充信息，如其位置或服务器本身（名称和版本等）的消息头，如 Server 头部。

<table>
<thead>
<tr>
<th>协议头</th>
<th>说明</th>
<th>举例</th>
</tr>
</thead>
<tbody>
<tr>
<td>Accept-Ranges</td>
<td>字段的值表示可用于定义范围的单位</td>
<td>Accept-Ranges: bytes</td>
</tr>
<tr>
<td>Age</td>
<td>创建响应的时间</td>
<td>Age：5744337</td>
</tr>
<tr>
<td>ETag</td>
<td>唯一标识分配的资源</td>
<td>Etag：W/"585cd998-7c0f"</td>
</tr>
<tr>
<td>Location</td>
<td>表示重定向后的 URL</td>
<td>Location: <a href="http://www.zcmhi.com/archives/94.html" target="_blank"><http://www.zcmhi.com/archives/94.html</a></td>>
</tr>
<tr>
<td>Retry-After</td>
<td>告知客户端多久后再发送请求</td>
<td>Retry-After: 120</td>
</tr>
<tr>
<td>Server</td>
<td>告知客户端服务器信息</td>
<td>Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)</td>
</tr>
<tr>
<td>Vary</td>
<td>缓存控制</td>
<td>Vary: Origin</td>
</tr>
</tbody>
</table>

#### 实体头部

请求/响应报文中实体部分的首部，包含有关实体主体的更多信息，比如主体长 (Content-Length) 度或其 MIME 类型，如 Content-Type 头部。

<table>
<thead>
<tr>
<th>协议头</th>
<th>说明</th>
<th>举例</th>
</tr>
</thead>
<tbody>
<tr>
<td>Allow</td>
<td>对某网络资源的有效的请求行为，不允许则返回 405</td>
<td>Allow: GET, HEAD</td>
</tr>
<tr>
<td>Content-encoding</td>
<td>返回内容的编码方式</td>
<td>Content-Encoding: gzip</td>
</tr>
<tr>
<td>Content-Length</td>
<td>返回内容的字节长度</td>
<td>Content-Length: 348</td>
</tr>
<tr>
<td>Content-Language</td>
<td>响应体的语言</td>
<td>Content-Language: en,zh</td>
</tr>
<tr>
<td>Content-Location</td>
<td>请求资源可替代的备用的另一地址</td>
<td>Content-Location: /index.htm</td>
</tr>
<tr>
<td>Content-MD5</td>
<td>返回资源的 MD5 校验值</td>
<td>Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==</td>
</tr>
<tr>
<td>Content-Range</td>
<td>在整个返回体中本部分的字节位置</td>
<td>Content-Range: bytes 21010-47021/47022</td>
</tr>
<tr>
<td>Content-Type</td>
<td>返回内容的 MIME 类型</td>
<td>Content-Type: text/html; charset=utf-8</td>
</tr>
<tr>
<td>Expires</td>
<td>响应过期的日期和时间</td>
<td>Expires: Thu, 01 Dec 2010 16:00:00 GMT</td>
</tr>
<tr>
<td>Last-Modified</td>
<td>请求资源的最后修改时间</td>
<td>Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT</td>
</tr>
</tbody>
</table>

### HTTP 连接

#### 持久连接

**非 Keep-alive**：早期 HTTP1.0，浏览器发起 http 请求需要与服务器建立新的 TCP 连接，请求处理后连接立即断开，重新请求重新连接。但每一个这样的连接，客户机和服务器都要分配 TCP 的缓冲区和变量，这给服务器带来的严重的负担。\
**Keep-alive**：HTTP1.1 默认持久连接，同一客户机可以连续请求通过相同的连接进行传送，一台服务器多个 web 页面也可通过单个 TCP 连接传送给同一个客户机。但长时间保持 TCP 连接会导致系统资源被无效占用。

##### 长连接 Or 短链接

[http的长连接和短连接（史上最通俗！）以及应用场景_luzhensmart的专栏-CSDN博客_长连接和短连接的使用场景](https://blog.csdn.net/luzhensmart/article/details/87186401)

**长连接**：多用于操作频繁，点对点的通讯，而且客户端连接数目较少的情况。例如即时通讯、网络游戏等。
**短连接**：用户数目较多的 Web 网站的 HTTP 服务一般用短连接。例如京东，淘宝这样的大型网站一般客户端数量达到千万级甚至上亿，若采用长连接势必会使得服务端大量的资源被无效占用，所以一般使用的是短连接。

#### 报文长度

长度在响应报文中有两种表现形式。

1. 对于小点的文件，直接给出 content-length，也就是本次返回的数据长度
2. 对于大文件，使用 Transfer-Encoding:chunked 字段，不传输数据长度，客户端只知道是分块传输，这也是订好了协议，客户端收到了会进行组装，每一个分块包含十六进制的长度值和数据，最后一个分块长度值为 0，表示实体结束，客户机可以以此为标志确认数据已经接收完毕。

曾经用 py 写过下载脚本，就利用了分块传输的方法：

```python
file_size = int(r.headers['content-length'])
with tqdm(total=file_size, unit='B', unit_scale=True, unit_divisor=1024, ascii=True,desc=filename) as bar:
	with open(file_path, 'wb') as fp:
		for chunk in r.iter_content(chunk_size=512):
		if chunk:
			fp.write(chunk)
			bar.update(len(chunk))
```

### HTTP 方法

HTTP/1.0 定义了三种请求方法：`GET`, `POST` 和 `HEAD` 方法。

HTTP/1.1 增加了六种请求方法：`OPTIONS`, `PUT`, `PATCH`, `DELETE`, `TRACE` 和 `CONNECT` 方法。

~~感觉实际生产中很少会用那 6 种方法，极大的复杂化了 api~~

<table>
<thead>
<tr>
<th>方法</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>GET</td>
<td>请求指定的页面信息，并返回具体内容，通常只用于读取数据。</td>
</tr>
<tr>
<td>HEAD</td>
<td>类似于 GET 请求，只不过返回的响应中没有具体的内容，用于获取报头。</td>
</tr>
<tr>
<td>POST</td>
<td>向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立或已有资源的更改。</td>
</tr>
<tr>
<td>PUT</td>
<td>替换指定的资源，没有的话就新增。</td>
</tr>
<tr>
<td>DELETE</td>
<td>请求服务器删除 URL 标识的资源数据。</td>
</tr>
<tr>
<td>CONNECT</td>
<td>将服务器作为代理，让服务器代替用户进行访问。</td>
</tr>
<tr>
<td>OPTIONS</td>
<td>向服务器发送该方法，会返回对指定资源所支持的 HTTP 请求方法。</td>
</tr>
<tr>
<td>TRACE</td>
<td>回显服务器收到的请求数据，即服务器返回自己收到的数据，主要用于测试和诊断。</td>
</tr>
<tr>
<td>PATCH</td>
<td>是对 PUT 方法的补充，用来对已知资源进行局部更新。</td>
</tr>
</tbody>
</table>

#### GET or POST

- get 提交的数据会放在 URL 之后，并且请求参数会被完整的保留在浏览器的记录里，由于参数直接暴露在 URL 中，可能会存在安全问题，因此往往用于获取资源信息。而 post 参数放在请求主体中，并且参数不会被保留，相比 get 方法，post 方法更安全，主要用于修改服务器上的资源。
- get 请求只支持 URL 编码，post 请求支持多种编码格式。
- get 只支持 ASCII 字符格式的参数，而 post 方法没有限制。
- get 提交的数据大小有限制（这里所说的限制是针对浏览器而言的），而 post 方法提交的数据没限制
- get 方式需要使用 Request.QueryString 来取得变量的值，而 post 方式通过 Request.Form 来获取。
- get 方法产生一个 TCP 数据包，post 方法产生两个（并不是所有的浏览器中都产生两个）

> 对于 GET 方式的请求，浏览器会把 http header 和 data 一并发送出去，服务端响应 200，请求成功。
> 对于 POST 方式的请求，浏览器会先发送 http header 给服务端，告诉服务端等一下会有数据过来，服务端响应 100 continue，告诉浏览器我已经准备接收数据，浏览器再 post 发送一个 data 给服务端，服务端响应 200，请求成功。

<table>
<thead>
<tr>
<th>操作方式</th>
<th>数据位置</th>
<th>明文密文</th>
<th>数据安全</th>
<th>长度限制</th>
<th>应用场景</th>
</tr>
</thead>
<tbody>
<tr>
<td>GET</td>
<td>HTTP 包头<br>如果数据是中文或其它字符，则进行 BASE64 编码。</td>
<td>明文</td>
<td>不安全</td>
<td>长度较小</td>
<td>查询数据</td>
</tr>
<tr>
<td>POST</td>
<td>HTTP 正文</td>
<td>可明可密</td>
<td>安全</td>
<td>支持较大数据传输</td>
<td>修改数据</td>
</tr>
</tbody>
</table>

**长度限制：**

- GET 方法是通过 URL 传递数据的，而 URL 本身并没有对数据的长度进行限制，真正限制 GET 长度的是浏览器，并且这个长度不是只针对数据部分，而是针对整个 URL 而言，在这之中，不同的服务器同样影响 URL 的最大长度限制。因此对于特定的浏览器，GET 的长度限制不同。
- POST 方法请求参数在请求主体中，理论上讲，post 方法是没有大小限制的，而真正起限制作用的是服务器处理程序的处理能力。

### HTTP 状态

HTTP 协议是**无连接无状态**的，并不保存关于客户端的任何信息。

通常有两种方法保持会话：

#### Session

服务器创建并保存键值对：SessionId-Session，然后将 SessionId 下发给客户端，客户端将其存在 Cookie 中，每次请求带上这个 SessionId，服务器就可以将状态和会话联系起来。

> 优点：安全性高，因为状态信息保存在服务器端。
> 缺点：由于大型网站往往采用的是分布式服务器，倘若同一个浏览器两次 HTTP 请求分别落在不同的服务器上时，基于 Session 的方法就不能实现会话保持了。
> 【解决方法】：采用中间件，例如 Redis，我们通过将 Session 的信息存储在 Redis 中，使得每个服务器都可以访问到之前的状态信息

#### Cookie

服务器发送响应消息时在响应头中设置 Set-Cookie 字段，存储客户端的状态信息。
客户端根据这个字段来创建 Cookie 并在请求时带上（每个 Cookie 都包含着客户端的状态信息），从而实现状态保持。

> 优点：服务器不用保存状态信息， 减轻服务器存储压力，同时便于服务端做水平拓展。
> 缺点：该方式不够安全，因为状态信息存储在客户端，这意味着不能在会话中保存机密数据；每次发起 HTTP 请求时都需要发送额外的 Cookie 到服务器端，会占用更多带宽。

#### Cookie 被禁用

若遇到 Cookie 被禁用的情况，则可以通过重写 URL 的方式将会话标识放在 URL 的参数里，也可以实现会话保持。

### HTTP 状态码

HTTP 状态码由三个十进制数字组成，第一个数字定义了状态码的类型，后两个并没有起到分类的作用。

HTTP 状态码共有 5 种类型：

<table>
<thead>
<tr>
<th>分类</th>
<th>分类描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>1XX</td>
<td>指示信息 -- 表示请求正在处理</td>
</tr>
<tr>
<td>2XX</td>
<td>成功 -- 表示请求已被成功处理完毕</td>
</tr>
<tr>
<td>3XX</td>
<td>重定向 -- 要完成的请求需要进行附加操作</td>
</tr>
<tr>
<td>4XX</td>
<td>客户端错误 -- 请求有语法错误或者请求无法实现，服务器无法处理请求</td>
</tr>
<tr>
<td>5XX</td>
<td>服务器端错误 -- 服务器处理请求出现错误</td>
</tr>
</tbody>
</table>

常见的状态码有如下几种：

- `200 OK` 客户端请求成功
- `204 No Content` 请求处理成功，但没有资源返回
- `206 Partial Content` 范围请求返回（Content-Range）
- `301 Moved Permanently` 请求永久重定向
- `302 Moved Temporarily` 请求临时重定向
- `304 Not Modified` 文件未修改，可以直接使用缓存的文件。
- `307 Temporary Redirect` 与 302 的区别：遵照浏览器标准，不会从 POST 变成 GET
- `400 Bad Request` 由于客户端请求有语法错误，不能被服务器所理解。
- `401 Unauthorized` 请求未经授权。这个状态代码必须和 WWW-Authenticate 报头域一起使用
- `403 Forbidden` 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
- `404 Not Found` 请求的资源不存在，例如，输入了错误的 URL
- `500 Internal Server Error` 服务器发生不可预期的错误，导致无法完成客户端的请求。
- `501 Not Implemented` 服务器不支持请求方法
- `502 Bad Gateway` 网关错误
- `503 Service Unavailable` 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。
- `504 Bad Gateway timeout` 网关超时
- `505 HTTP Version Not Supported` 不支持的 HTTP 版本

> 更多可见 [HTTP 状态码_w3cschool](https://www.w3cschool.cn/http/g9prxfmx.html)

#### 面试对状态码常见问法

##### 状态码 301 和 302 的区别

- 301：永久移动。请求的资源已被永久的移动到新的 URI，旧的地址已经被永久的删除了。返回信息会包括新的 URI，浏览器会自动定向到新的 URI。今后新的请求都应使用新的 URI 代替。
- 302：临时移动。与 301 类似，客户端拿到服务端的响应消息后会跳转到一个新的 URL 地址。但资源只是临时被移动，旧的地址还在，客户端应继续使用原有 URI。

##### HTTP 异常状态码

该问题一般只需要回答 3, 4 , 5 开头的一些常见异常状态码即可。

## HTTPS

HTTPS 即 HTTP over TLS，是一种在加密信道进行 HTTP 内容传输的协议。

### 加密方式

HTTPS 采用对称加密和非对称加密相结合的方式：

首先使用 SSL/TLS 协议进行加密传输，为了弥补非对称加密的缺点，HTTPS 采用证书来进一步加强非对称加密的安全性。
通过非对称加密，客户端和服务端协商好之后进行通信传输的对称密钥，后续的所有信息都通过该对称秘钥进行加密解密，完成整个 HTTPS 的流程。

### 工作方式

<table>
	<tr>
	<td>协议</td>
	<td>特点</td>
	<td>工作方式</td>
	</tr>
	<tr>
		<td>HTTP</td>
		<td>
		<li>明文传输，数据未加密，安全性较差
		<li>默认 80 端口
		<li>3 次握手建立连接
		</td>
		<td>
		<ol>
		<li>客户端请求服务器 80 端口，建立 TCP 连接
		<li>客户端从套接字接口发送 HTTP 请求报文和接收 HTTP 响应报文
		<li>服务端从套接字接口接收 HTTP 请求报文和发送 HTTP 响应报文
		<li>通信结束，客户端与服务器关闭连接
		</ol>
		</td>
	</tr>
	<tr>
		<td>HTTPS</td>
		<td>
		<li>加密，安全性较好
		<li>默认 443 端口
		<li>需数字认证机构（Certificate Authority, CA）的证书
		<li>除 TCP 的 3 次握手，还需要 SSL 协商
		</td>
		 <td>
		<ol>
		<li>客户端请求服务器 443 端口，建立 TCP 连接（包括支持算法，密钥长度等）
		<li>服务端从双方共同支持的加密算法列表中选择一种返回给客户端（包括密钥组件）
		<li>服务器返回自身 CA 证书的报文（包含证书的颁发机构、过期时间、服务端的公钥等信息）
		<li>服务端发送一个完成报文通知客户端 SSL 的第一阶段已经<b>协商完成</b>
		<li>客户端用本地证书库的根证书校验 CA 证书，生成随机密码串，用公钥加密发送给服务器，即回应报文
		<li>紧接着客户端会发送一个报文提示服务端在此之后的报文是采用密码串加密的
		<li>客户端发送一个 finish 报文（包含第一次握手至今所有报文的整体校验值）
		<li>服务端同样发送与第 6 步中相同作用的报文，最后发送 finish 报文告诉客户端自己能够正确解密报文
		<li>SSL 连接建立
		</ol>
		</td>
	</tr>
</table>

> CA 证书防止的方式：（为什么可以信任 CA 证书）
> 篡改：加密签名与原文签名对比
> 调包：请求域名与证书域名对比

另附 HTTP 版本演变：

> 直接造访 [HTTP x.x](https://leetcode-cn.com/leetbook/read/networks-interview-highlights/ez4zv6/)

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112161819417.png" style="zoom:67%;" />

## DNS

DNS（Domain Name System）是域名系统的英文缩写，是一种组织成域层次结构的计算机和网络服务命名系统，提供了主机名和 IP 地址之间相互转换的服务。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112161824712.png" style="zoom:80%;" />

### 查询方式

- **递归查询**：如果主机所询问的本地域名服务器不知道被查询域名的 IP 地址，那么本地域名服务器就以 DNS 客户端的身份，向其他根域名服务器继续发出查询请求报文，即替主机继续查询，而不是让主机自己进行下一步查询。**帮你查**
- **迭代查询**：当根域名服务器收到本地域名服务器发出的迭代查询请求报文时，要么给出所要查询的 IP 地址，要么告诉本地服务器下一步应该找哪个域名服务器进行查询，然后让本地服务器进行后续的查询。**自己查**

### 传输方式

DNS 可以使用 UDP 或者 TCP 进行传输，使用的端口号都为 53。

大多数情况下 DNS 使用 UDP 进行传输，不需要经过 TCP 三次握手的过程，从而大大提高了响应速度。但要求域名解析器和域名服务器都必须自己处理超时和重传从而保证可靠性。

在两种情况下会使用 TCP 进行传输：

- 如果返回的响应超过的 512 字节（UDP 最大只支持 512 字节的数据）。
- 区域传送（区域传送是主域名服务器向辅助域名服务器传送变化的那部分数据

> 因为数据同步传送的数据量比一个请求和应答的数据量要多，而 TCP 允许的报文长度更长，因此为了保证数据的正确性，会使用基于可靠连接的 TCP。

### DNS 劫持

DNS 劫持即域名劫持，是通过将原域名对应的 IP 地址进行替换从而使得用户访问到错误的网站或者使得用户无法正常访问网站的一种攻击方式。

域名劫持往往只能在特定的网络范围内进行，范围外的 DNS 服务器能够返回正常的 IP 地址。

#### 预防手段

- 直接使用 IP 访问
- 直接指定 DNS 服务器（如谷歌的 8.8.8.8）

## More

### Socket 套接字

> Socket 是对 TCP/IP 协议族的一种封装，是应用层与 TCP/IP 协议族通信的中间软件抽象层。从设计模式的角度看来，Socket 其实就是一个门面模式，它把复杂的 TCP/IP 协议族隐藏在 Socket 接口后面，对用户来说，一组简单的接口就是全部，让 Socket 去组织数据，以符合指定的协议。
>
> Socket 还可以认为是一种网络间不同计算机上的进程通信的一种方法，利用三元组（ip 地址，协议，端口）就可以唯一标识网络中的进程，网络中的进程通信可以利用这个标志与其它进程进行交互。
>
> Socket 起源于 Unix ，Unix/Linux 基本哲学之一就是“一切皆文件”，都可以用“打开 (open) –> 读写 (write/read) –> 关闭 (close)”模式来进行操作。因此 Socket 也被处理为一种特殊的文件。

套接字（Socket）是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象，网络进程通信的一端就是一个套接字，不同主机上的进程便是通过套接字发送报文来进行通信。
例如 TCP 用主机的 IP 地址 + 端口号作为 TCP 连接的端点，这个端点就叫做套接字。

套接字主要有以下三种类型：

- 流套接字（SOCK_STREAM）：流套接字基于 TCP 传输协议，主要用于提供面向连接、可靠的数据传输服务。由于 TCP 协议的特点，使用流套接字进行通信时能够保证数据无差错、无重复传送，并按顺序接收，通信双方不需要在程序中进行相应的处理。
- 数据报套接字（SOCK_DGRAM）：和流套接字不同，数据报套接字基于 UDP 传输协议，对应于无连接的 UDP 服务应用。该服务并不能保证数据传输的可靠性，也无法保证对端能够顺序接收到数据。此外，通信两端不需建立长时间的连接关系，当 UDP 客户端发送一个数据给服务器后，其可以通过同一个套接字给另一个服务器发送数据。当用 UDP 套接字时，丢包等问题需要在程序中进行处理。
- 原始套接字（SOCK_RAW）：由于流套接字和数据报套接字只能读取 TCP 和 UDP 协议的数据，当需要传送非传输层数据包（例如 Ping 命令时用的 ICMP 协议数据包）或者遇到操作系统无法处理的数据包时，此时就需要建立原始套接字来发送。

### URI 和 URL

- URL，即统一资源定位符 ( *Uniform Resource Locator* )，URL 其实就是我们平时上网时输入的网址，它标识一个互联网资源，并指定对其进行操作或获取该资源的方法。
- URI，即统一资源标识符（ *Uniform Resource Identifier* ），只要能唯一标识资源的就是 URI，在 URI 的基础上给出其资源的访问方式的就是 URL

### 抓包工具

- 其实就是将抓包工具视为中间人，其对于本地而言相当于服务端；而对于真正的服务端而言则相当于客户端；
- 抓包工具分别和本地以及服务器都进行 TLS 握手协商；
- 这就需要本地能够信任抓包工具提供的证书（也就是需要额外安装一个证书）

### 网页解析全过程

> 这部分可以看看：[当···时发生了什么？what-happens-when](https://github.com/skyline75489/what-happens-when-zh_CN)

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112161850389.png" style="zoom:67%;" />

1. **DNS 解析**：当用户输入一个网址并按下回车键的时候，浏览器获得一个域名，而在实际通信过程中，我们需要的是一个 IP 地址，因此我们需要先把域名转换成相应 IP 地址。
2. **TCP 连接**：浏览器通过 DNS 获取到 Web 服务器真正的 IP 地址后，便向 Web 服务器发起 TCP 连接请求，通过 TCP 三次握手建立好连接后，浏览器便可以将 HTTP 请求数据发送给服务器了
3. **发送 HTTP 请求**：浏览器向 Web 服务器发起一个 HTTP 请求，HTTP 协议是建立在 TCP 协议之上的应用层协议，其本质是在建立起的 TCP 连接中，按照 HTTP 协议标准发送一个索要网页的请求。在这一过程中，会涉及到负载均衡等操作。

> 负载均衡，英文名为 Load Balance，其含义是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，例如 FTP 服务器、Web 服务器、企业核心服务器和其他主要任务服务器等，从而协同完成工作任务。负载均衡建立在现有的网络之上，它提供了一种透明且廉价有效的方法扩展服务器和网络设备的带宽、增加吞吐量、加强网络处理能力并提高网络的灵活性和可用性。
> 负载均衡是分布式系统架构设计中必须考虑的因素之一，例如天猫、京东等大型用户网站中为了处理海量用户发起的请求，其往往采用分布式服务器，并通过引入反向代理等方式将用户请求均匀分发到每个服务器上，而这一过程所实现的就是负载均衡。

4. **处理请求并返回**：服务器获取到客户端的 HTTP 请求后，会根据 HTTP 请求中的内容来决定如何获取相应的文件，并将文件发送给浏览器。
5. **浏览器渲染**：浏览器根据响应开始显示页面，首先解析 HTML 文件构建 DOM 树，然后解析 CSS 文件构建渲染树，等到渲染树构建完成后，浏览器开始布局渲染树并将其绘制到屏幕上。
6. **断开连接**：客户端和服务器通过四次挥手终止 TCP 连接。
