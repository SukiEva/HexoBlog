---
title: 《图解HTTP》笔记
tags:
  - HTTP
categories: [Computer Basics,Computer Network]
toc: true
date: 2022-03-06 13:36
updated: 2022-03-06 13:36
---

> 本书图文并茂，大量图片穿插文中，生动形象地向读者介绍每一个应用案例，减少了读者阅读时的枯燥感。
>
> 借助一张张配图，读者们不仅会加深视觉记忆，在轻松愉悦的氛围中，还可以更深刻地理解通信机 制等背后的工作原理。
>
> 正所谓一图胜千文。

<!-- more -->

## HTTP 协议

### 请求/响应通信

请求报文由**请求方法、请求 URI、协议版本、可选的请求首部字段和内容实体**构成。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061347720.png" style="zoom:80%;" />

响应报文由**协议版本、状态码（原因短语）、可选的响应首部字段和主体**构成。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061349516.png" style="zoom:80%;" />

### 不保存状态

HTTP 是一种不保存状态，即无状态（stateless）协议。

HTTP 协议自身不对请求和响应之间的通信状态进行保存，协议对于发送过的请求或响应都不做持久化处理。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061351286.png" style="zoom: 80%;" />

### HTTP 方法

#### GET：获取资源

GET 方法用来请求访问已被 URI 识别的资源。

指定的资源经服务器端解析后返回响应内容。

> 如果请求的资源是文本，那就保持原样返回；
>
> 如果是像 CGI（Common Gateway Interface，通用网关接 口）那样的程序，则返回经过执行后的输出结果。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061355943.png" style="zoom: 80%;" />

#### POST：传输实体主体

POST 方法用来传输实体的主体。

虽然用 GET 方法也可以传输实体的主体，但一般不用 GET 方法进行 传输，而是用 POST 方法。

虽说 POST 的功能与 GET 很相似，但 POST 的主要目的并不是获取响应的主体内容。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061356954.png" style="zoom: 80%;" />

#### PUT：传输文件

PUT 方法用来传输文件。

就像 FTP 协议的文件上传一样，要求在请求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置。

但是，鉴于 HTTP/1.1 的 PUT 方法自身不带验证机制，任何人都可以上传文件 , 存在安全性问题，因此一般的 Web 网站不使用该方法。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061359052.png" style="zoom: 80%;" />

#### HEAD：获得报文首部

HEAD 方法和 GET 方法一样，只是不返回报文主体部分。

用于确认 URI 的有效性及资源更新的日期时间等。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061359184.png" style="zoom: 80%;" />

#### DELETE：删除文件

DELETE 方法用来删除文件，是与 PUT 相反的方法。

DELETE 方法按请求 URI 删除指定的资源。

但是，HTTP/1.1 的 DELETE 方法本身和 PUT 方法一样不带验证机制，所以一般的 Web 网站也不使用 DELETE 方法。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061400518.png" style="zoom:80%;" />

#### OPTIONS：询问支持的方法

OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061406769.png" style="zoom: 80%;" />

#### TRACE：追踪路径

TRACE 方法是让 Web 服务器端将之前的请求通信环回给客户端的方法。

发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器端就将该数字减 1，当数值刚好减到 0 时，就停止继续传输，最后接收到请求的服务器端则返回状态码 200 OK 的响应。

但是，TRACE 方法本来就不怎么常用，再加上它容易引发 XST（Cross-Site Tracing，跨站追踪）攻击，通常就更不会用到了。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061402888.png" style="zoom: 80%;" />

#### CONNECT：要求用隧道协议连接代理

CONNECT 方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信。

主要使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。

格式：`CONNECT 代理服务器名: 端口号 HTTP 版本`

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061404359.png" style="zoom: 80%;" />

### 持久连接

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061407328.png" alt="image-20220306140731254" style="zoom: 67%;" />

为解决上述 TCP 连接的问题，HTTP/1.1 和一部分的 HTTP/1.0 想出了持久连接（HTTP Persistent Connections，也称为 HTTP keep-alive 或 HTTP connection reuse）的方法。

持久连接的特点是，**只要任意一端没有明确提出断开连接，则保持 TCP 连接状态**。

在 HTTP/1.1 中，所有的连接默认都是持久连接。

#### 管线化 Pipelining

持久连接使得多数请求以管线化（pipelining）方式发送成为可能。

从前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术出现后，不用等待响应亦可直接发送下一个请求。

这样就能够做到同时**并行发送多个请求**，而不需要一个接一个地等待响应了。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061410650.png" style="zoom: 67%;" />

### 使用 COOKIE

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061412524.png" style="zoom: 67%;" />

保留无状态协议这个特征的同时又要解决类似的矛盾问题，于是引入了 Cookie 技术。Cookie 技术通过在请求和响应报文中写入 Cookie 信息来控制客户端的状态。

- Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie 的首部字段信息，通知客户端保存 Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入 Cookie 值后发送出去。
- 服务器端发现客户端发送过来的 Cookie 后，会去检查究竟是从哪一个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前的状态信息。

<table>
	<tr>
		<td><img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061414206.png"  /></td>
		<td><img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061415535.png"  /></td>
	</tr>
</table>

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061417981.png" style="zoom: 67%;" />

## HTTP 报文

用于 HTTP 协议交互的信息被称为 HTTP 报文。请求端（客户端）的 HTTP 报文叫做请求报文，响应端（服务器端）的叫做响应报文。

HTTP 报文本身是由多行（用 CR+LF 作换行符）数据构成的字符串文本。

HTTP 报文大致可分为报文首部和报文主体两块。两者由最初出现的 空行（CR+LF）来划分。通常，并不一定要有报文主体。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061435574.png" style="zoom:67%;" />

- 请求行包含用于请求的方法，请求 URI 和 HTTP 版本。
- 状态行包含表明响应结果的状态码，原因短语和 HTTP 版本。
- 首部字段包含表示请求和响应的各种条件和属性的各类首部。

### 编码传输

HTTP 在传输数据时可以按照数据原貌直接传输，但也可以在传输过 程中通过编码提升传输速率。但是，编码的操作需要计算机来完成，因此会消耗更多的 CPU 等资源。

#### 报文/实体主体

- 报文（message）：HTTP 通信中的基本单位，由 8 位组字节流（octet sequence， 其中 octet 为 8 个比特）组成，通过 HTTP 通信传输。
- 实体（entity）：作为请求或响应的有效载荷数据（补充项）被传输，其内容由实体首部和实体主体组成。

HTTP 报文的主体用于传输请求或响应的**实体主体**。

> 通常，报文主体等于实体主体。
>
> 只有当传输中进行编码操作时，实体主体的**内容发生变化**，才导致它和报文主体产生差异。

#### 压缩传输的内容编码

向待发送邮件内增加附件时，为了使邮件容量变小，我们会先用 ZIP 压缩文件之后再添加附件发送。

HTTP 协议中有一种被称为内容编码的功能也能进行类似的操作。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061442983.png" style="zoom:67%;" />

常用的内容编码有以下几种：

- gzip（GNU zip）
- compress（UNIX 系统的标准压缩）
- deflate（zlib）
- identity（不进行编码）

#### 分割发送的分块传输编码

在传输大容量数据时，通过把数据分割成多块，能够让浏览器逐步显示页面。

这种把实体主体分块的功能称为分块传输编码（Chunked Transfer Coding）。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061443064.png" style="zoom:67%;" />

- 分块传输编码会将实体主体分成多个部分（块）。每一块都会用十六 进制来标记块的大小，而实体主体的最后一块会使用“0(CR+LF)”来标记。
- 使用分块传输编码的实体主体会由接收的客户端负责解码，恢复到编码前的实体主体。

HTTP/1.1 中存在一种称为传输编码（Transfer Coding）的机制，它可以在通信时按某种编码方式传输，但只定义作用于分块传输编码中。

### 对象集合

发送邮件时，我们可以在邮件里写入文字并添加多份附件。

这是因为采用了 MIME（Multipurpose Internet Mail Extensions，多用途因特网邮件扩展）机制，它允许邮件处理文本、图片、视频等多个不同类型的数据。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061445336.png" style="zoom:80%;" />

HTTP 协议中也采纳了多部分对象集合（Multipart），发送的一份报文主体内可含有多类型实体。通常是在图片或文本文件等上传时使用。

在 HTTP 报文中使用多部分对象集合时，需要在首部字段里加上 Content-type。

### 获取部分内容的范围请求

如果下载过程中遇到网络中断的情况，那就必须重头开始。为了解决上述问题，需要一种可恢复的机制。所谓恢复是指能从之前下载中断处恢复下载。

要实现该功能需要指定下载的实体范围。像这样，指定范围发送的请求叫做范围请求（Range Request）。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203061450934.png" style="zoom: 67%;" />

执行范围请求时，会用到首部字段 Range 来指定资源的 byte 范围。

> 针对范围请求，响应会返回状态码为 206 Partial Content 的响应报文。
>
> 另外，对于多重范围的范围请求，响应会在首部字段 ContentType 标明 multipart/byteranges 后返回响应报文。

## HTTP 协作

### 通信数据转发

HTTP 通信时，除客户端和服务器以外，还有一些用于通信数据转发的应用程序，例如代理、网关和隧道。它们可以配合服务器工作。

#### 代理

代理是一种有转发功能的应用程序，它扮演了位于服务器和客户端“中间人”的角色，接收由客户端发送的请求并转发给服务器，同时也接收服务器返回的响应并转发给客户端。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062113232.png" style="zoom:80%;" />

使用代理服务器的理由有：利用缓存技术（稍后讲解）减少网络带宽的流量，组织内部针对特定网站的访问控制，以获取访问日志为主要目的等等。

代理有多种使用方法，按两种基准分类。一种是是否使用缓存，另一种是是否会修改报文。

- 缓存代理：代理转发响应时，缓存代理（Caching Proxy）会预先将资源的副本（缓存）保存在代理服务器上。当代理再次接收到对相同资源的请求时，就可以不从源服务器那里获取资源，而是将之前缓存的资源作为响应返回。
- 透明代理：转发请求或响应时，不对报文做任何加工的代理类型被称为透明代理（Transparent Proxy）。反之，对报文内容进行加工的代理被称为非透明代理。

#### 网关

网关是转发其他服务器通信数据的服务器，接收从客户端发送来的请求时，它就像自己拥有资源的源服务器一样对请求进行处理。有时客户端可能都不会察觉，自己的通信目标是一个网关。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062135522.png" style="zoom:80%;" />

网关的工作机制和代理十分相似。而网关能使通信线路上的服务器提供非 HTTP 协议服务。

利用网关能提高通信的安全性，因为可以在客户端与网关之间的通信线路上加密以确保连接的安全。

比如，网关可以连接数据库，使用 SQL 语句查询数据。另外，在 Web 购物网站上进行信用卡结算时，网关可以和信用卡结算系统联动。

#### 隧道

隧道是在相隔甚远的客户端和服务器两者之间进行中转，并保持双方通信连接的应用程序。

隧道可按要求建立起一条与其他服务器的通信线路，届时使用 SSL 等加密手段进行通信。隧道的目的是确保客户端能与服务器进行安全的通信。

隧道本身不会去解析 HTTP 请求。也就是说，请求保持原样中转给之 后的服务器。隧道会在通信双方断开连接时结束。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062138636.png" style="zoom:80%;" />

### 保存资源的缓存

缓存是指代理服务器或客户端本地磁盘内保存的资源副本。利用缓存可减少对源服务器的访问，因此也就节省了通信流量和通信时间。

缓存服务器是代理服务器的一种，并归类在缓存代理类型中。换句话说，当代理转发从服务器返回的响应时，代理服务器将会保存一份资源的副本。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062144618.png" style="zoom:67%;" />

缓存服务器的优势在于利用缓存可避免多次从源服务器转发资源。因此客户端可就近从缓存服务器上获取资源，而源服务器也不必多次处理相同的请求了。

## HTTPS

### HTTP 缺点

HTTP 主要有这些不足：

- 通信使用明文（不加密），内容可能会被窃听
- 不验证通信方的身份，因此有可能遭遇伪装
- 无法证明报文的完整性，所以有可能已遭篡改

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062156625.png" style="zoom:80%;" />

### HTTP + 加密 + 认证 + 完整性保护 = HTTPS

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062201021.png" style="zoom:80%;" />

#### HTTPS 是身披 SSL 外壳的 HTTP

HTTPS 并非是应用层的一种新协议。只是 HTTP 通信接口部分用 SSL（Secure Socket Layer）和 TLS（Transport Layer Security）协议代替而已。

通常，HTTP 直接和 TCP 通信。当使用 SSL 时，则演变成先和 SSL 通信，再由 SSL 和 TCP 通信了。简言之，所谓 HTTPS，其实就是身披 SSL 协议这层外壳的 HTTP。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062217085.png" style="zoom: 80%;" />

> SSL 是独立于 HTTP 的协议，所以不光是 HTTP 协议，其他运行在应用层的 SMTP 和 Telnet 等协议均可配合 SSL 协议使用。
>
> 可以说 SSL 是 当今世界上应用最为广泛的网络安全技术。

#### 相互交换密钥的公开密钥加密技术

SSL 采用一种叫做公开密钥加密（Public-key cryptography）的加密处理方式。

近代的加密方法中加密算法是公开的，而密钥却是保密的。通过这种方式得以保持加密方法的安全性。

> 对称加密：加密和解密同用一个密钥
>
> <img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062221048.png" style="zoom:80%;" />
>
> 非对称加密：公开密钥加密使用一对非对称的密钥。一把叫做私有密钥（private key），另一把叫做公开密钥（public key）
>
> <img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062223969.png" style="zoom:67%;" />

HTTPS 采用共享密钥加密和公开密钥加密两者并用的混合加密机制。

若密钥能够实现安全交换，那么有可能会考虑仅使用公开密钥加密来通信。但是公开密钥加密与共享密钥加密相比，其处理速度要慢。

所以应充分利用两者各自的优势，将多种方法组合起来用于通信。在交换密钥环节使用公开密钥加密方式，之后的建立通信交 换报文阶段则使用共享密钥加密方式。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062225654.png" style="zoom: 80%;" />

#### 证明公开密钥正确性的证书

遗憾的是，公开密钥加密方式还是存在一些问题的。那就是无法证明公开密钥本身就是货真价实的公开密钥。或许在公开密钥传输途中，真正的公开密钥已经被攻击者替换掉了。

为了解决上述问题，可以使用由数字证书认证机构（CA，Certificate Authority）和其相关机关颁发的公开密钥证书。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062230419.png" style="zoom:80%;" />

### HTTPS 通信

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062235169.png" style="zoom:80%;" />

- 步骤 1： 客户端通过发送 Client Hello 报文开始 SSL 通信。报文中包含客户端支持的 SSL 的指定版本、加密组件（Cipher Suite）列表（所使用的加密算法及密钥长度等）。
- 步骤 2： 服务器可进行 SSL 通信时，会以 Server Hello 报文作为应答。和客户端一样，在报文中包含 SSL 版本以及加密组件。服务器的加密组件内容是从接收到的客户端加密组件内筛选出来的。
- 步骤 3： 之后服务器发送 Certificate 报文。报文中包含公开密钥证书。
- 步骤 4： 最后服务器发送 Server Hello Done 报文通知客户端，最初阶段的 SSL 握手协商部分结束。
- 步骤 5： SSL 第一次握手结束之后，客户端以 Client Key Exchange 报 文作为回应。报文中包含通信加密中使用的一种被称为 Pre-master secret 的随机密码串。该报文已用步骤 3 中的公开密钥进行加密。
- 步骤 6： 接着客户端继续发送 Change Cipher Spec 报文。该报文会提示服务器，在此报文之后的通信会采用 Pre-master secret 密钥加密。
- 步骤 7： 客户端发送 Finished 报文。该报文包含连接至今全部报文的整体校验值。这次握手协商是否能够成功，要以服务器是否能够正确解密该报文作为判定标准。
- 步骤 8： 服务器同样发送 Change Cipher Spec 报文。
- 步骤 9： 服务器同样发送 Finished 报文。
- 步骤 10： 服务器和客户端的 Finished 报文交换完毕之后，SSL 连接 就算建立完成。当然，通信会受到 SSL 的保护。从此处开始进行应用层协议的通信，即发送 HTTP 请求。
- 步骤 11： 应用层协议通信，即发送 HTTP 响应。
- 步骤 12： 最后由客户端断开连接。断开连接时，发送 close_notify 报文。上图做了一些省略，这步之后再发送 TCP FIN 报文来关闭与 TCP 的通信。

### HTTPS 缺点

HTTPS 也存在一些问题，那就是当使用 SSL 时，它的处理速度会变慢。

SSL 的慢分两种：一种是指通信慢，另一种是指由于大量消耗 CPU 及内存等资源，导致处理速度变慢。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203062239123.png" style="zoom:80%;" />

> 如果是非敏感信息则使用 HTTP 通信，只有在包含个人信息等敏感数据时，才利用 HTTPS 加密通信。
>
> 特别是每当那些访问量较多的 Web 网站在进行加密处理时，它们所承担着的负载不容小觑。在进行加密处理时，并非对所有内容都进行加密处理，而是仅在那些需要信息隐藏时才会加密，以节约资源。
>
> 除此之外，想要节约购买证书的开销也是原因之一。

## Web 攻击技术

简单的 HTTP 协议本身并不存在安全性问题，因此协议本身几乎不会 成为攻击的对象。

应用 HTTP 协议的服务器和客户端，以及运行在服务器上的 Web 应用等资源才是攻击目标。

> 从整体上看，HTTP 就是一个通用的单纯协议机制。因此它具备较多优势，但是在安全性方面则呈劣势。
>
> - 拿远程登录时会用到的 SSH 协议来说，SSH 具备协议级别的认证及会话管理等功能，HTTP 协议则没有。另外在架设 SSH 服务方面，任何人都可以轻易地创建安全等级高的服务，而 HTTP 即使已架设好服务器，但若想提供服务器基础上的 Web 应用，很多情况下都需要重新开发。
> - 因此，开发者需要**自行设计并开发认证及会话管理功能来满足 Web 应用的安全**。而自行设计就意味着会出现各种形形色色的实现。结果，安全等级并不完备，可仍在运作的 Web 应用背后却隐藏着各种容易被攻击者滥用的安全漏洞的 Bug。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203121042239.png" style="zoom:80%;" />

### 攻击模式

- **以服务器为目标的主动攻击（active attack）**：攻击者通过直接访问 Web 应用， 把攻击代码传入的攻击模式。由于该模式是直接针对服务器上的资源进行攻击，因此攻击者需要能够访问到那些资源。
	- SQL 注入攻击
	- OS 命令注入攻击

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203121044184.png" style="zoom:80%;" />

- **以服务器为目标的被动攻击（passive attack）**：利用圈套策略执行攻击代码的攻击模式。在被动攻击过程中，攻击者不直接对目标 Web 应用访问发起攻击。
	- 跨站脚本攻击
	- 跨站点请求伪造

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202203121046510.png" style="zoom:80%;" />
