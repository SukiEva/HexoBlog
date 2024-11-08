---
title: 传输层
toc: true
date: 2021-12-21 19:17
tags:
  - Network
categories: [Computer Basics,Computer Network]
updated: 2021-12-21 19:17
references:
  - title: LeetCode
    url: https://leetcode-cn.com/leetbook/read/networks-interview-highlights/esegch/
  - title: 王道考研
    url: http://cskaoyan.com/forum.php
---

**传输层**（**Transport Layer**）位于 OSI 模型第四层。该层的协议为应用进程提供端到端的通信服务。它提供面向连接的数据流支持、可靠性、流量控制、多路复用等服务。

功能：使用网络层服务，为应用层提供服务

- 提供进程和进程之间的逻辑通信
- 复用和分用
- 传输层对收到的报文进行差错检测

传输层的重要协议：

- TCP（Transmission Control Protocol）传输控制协议
- UDP（User Datagram Protocol）用户数据报协议

<!-- more -->

## TCP

### 报文

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112212129515.png" style="zoom:33%;" />

### 三次握手

三次握手是 TCP 连接的建立过程。在握手之前，主动打开连接的客户端结束 CLOSE 阶段，被动打开的服务器也结束 CLOSE 阶段，并进入 LISTEN 阶段。随后进入三次握手阶段：

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112211934483.png" style="zoom: 40%;" />

> 标志位：SYN（Synchronize）、ACK（ACKnowledge Character）
> 序号：seq
> 确认号：ack
>
> 第 2 次握手为什么还要传回 SYN：
>
> ACK 是为了告诉客户端发来的数据已经接收无误，而传回 SYN 是为了把自己的初始序列号（seq）同步给客户端。

#### 为什么三次握手

三次握手保证两点：

- 保证双方都是双工通信：
	- 第一次握手，服务端确定客户端的发送正常
	- 第二次握手，客户端确认服务端的收发正常
	- 第三次握手，服务端确定客服端接收正常
- 如果只有第二次握手，服务端发给客服端的包丢了之后：
	- 服务端直接建立了连接，端口就会一直开着
	- 等到客户端因超时重新发出请求时，服务器就会重新开启一个端口连接
	- 端口越来越多，造成服务器开销的浪费

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112211949356.png" alt="image-20211221194951070" style="zoom: 33%;" />

#### 握手异常

| 异常                | 如何处理                                                                                                                                                                                   | 备注                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| 第一次握手 SYN 包丢失     | **服务端**不会进行任何相应的动作<br>**客户端**在一段时间内没有收到服务器发来的确认报文， 会等待一段时间后重新发送 SYN 同步报文，<br>若仍没有回应，则重复上述直到发送次数超过最大重传次数限制后，建立连接的系统调用会返回 -1                                                             | 客户端超时重传最大次数：`tcp_syn_retries`，默认 5 次（Linux 3.7 后为 6 次） |
| 第二次握手 SYN、ACK 包丢失 | **客户端**会采取第一次握手失败时的动作（超时重传）<br>**服务端**此时将阻塞在 accept() 系统调用处等待 client 再次发送 ACK 报文（同样超时重传）                                                                                               | 服务端超时重传最大次数：`tcp_synack_retries`，默认 5 次              |
| 第三次握手 ACK 包丢失     | 两次握手成功，**客户端**进入 `ESTABLISHED` 状态，**服务端**进入 `SYN_REC` 状态<br>**服务端**收不到 ACK，就一直重传 SYN、ACK 包，直到超过最大次数，断开 TCP 连接<br>**客户端**认为自己连接成功，开始向服务器端发送数据，服务端收到来自客户端发送来的数据时会发送 RST 报文给客户端，消除客户端单方面建立连接的状态 |                                                    |

### 四次挥手

四次挥手即 TCP 连接的释放，这里假设客户端主动释放连接。在挥手之前主动释放连接的客户端结束 ESTABLISHED 阶段，随后开始四次挥手：

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112211951252.png" style="zoom: 40%;" />

> 标志位：Fin（Finish）
> 阶段：FIN-WAIT（半关闭）
>
> CLOSE-WAIT：是服务端发出第一次挥手（整体第二次）进入的状态
>
> 表示“我准备关闭了，但是还有自己的事情处理一下，你等我处理完”
> 等服务器处理好自己的数据业务，则表示“我准备好了”，再发送 FIN 包
>
> TIME-WAIT：是第四次挥手后，客户端进入的状态，是客户端必要的等待时间。
>
> 目的是：等待服务端的对应端口关闭与客户端发送到服务端的数据到达（可能出现延迟）
>
> 如果不存在这个步骤就会导致两个问题:
>
> - 客户端立即关闭后，立即又用同样的端口握手并建立通信，此时上次的连接残留的数据包会被误认为是本次的，造成数据异常
>
> - 客户端直接关闭后，若服务端重新发送 FIN 包，客户端就会回应 RST，会报异常，但是实际没有问题的
>
> MSL（Maximum Segment Lifetime）：指一段 TCP 报文在传输过程中的最大生命周期
>
> 2 MSL 即是服务器端发出 FIN 报文和客户端发出的 ACK 确认报文所能保持有效的最大时长，为的是确认服务器能否接收到客户端发出的 ACK 确认报文

#### 为什么四次挥手

简单来说就是因为 TCP 是全双工的，两个方向的连接需要单独关闭。

FIN 释放连接报文和 ACK 确认接收报文是分别在两次握手中传输的：

- 当主动方在数据传送结束后发出连接释放的通知，由于被动方可能还有必要的数据要处理，所以会先返回 ACK 确认收到报文
- 当被动方也没有数据再发送的时候，则发出连接释放通知，对方确认后才完全关闭 TCP 连接。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112212000734.png" style="zoom:33%;" />

### 可靠传输

TCP 如何保证可靠传输：

- **数据分块**：应用数据被分割成 TCP 认为最适合发送的数据块。
- **序列号和确认应答**：TCP 给发送的每一个包进行编号，在传输的过程中，每次接收方收到数据后，都会对传输方进行确认应答，即发送 ACK 报文，这个 ACK 报文当中带有对应的确认序列号，告诉发送方成功接收了哪些数据以及下一次的数据从哪里开始发。除此之外，接收方可以根据序列号对数据包进行排序，把有序数据传送给应用层，并丢弃重复的数据。
- **校验和**：TCP 将保持它首部和数据部分的检验和。这是一个端到端的检验和，目的是检测数据在传输过程中的任何变化。如果收到报文段的检验和有差错，TCP 将丢弃这个报文段并且不确认收到此报文段。
- **流量控制**：TCP 连接的双方都有一个固定大小的缓冲空间，发送方发送的数据量不能超过接收端缓冲区的大小。当接收方来不及处理发送方的数据，会提示发送方降低发送的速率，防止产生丢包。TCP 通过滑动窗口协议来支持流量控制机制。
- **拥塞控制**：当网络某个节点发生拥塞时，减少数据的发送。
- **ARQ 协议**：也是为了实现可靠传输的，它的基本原理就是每发完一个分组就停止发送，等待对方确认。在收到确认后再发下一个分组。
- **超时重传**：当 TCP 发出一个报文段后，它启动一个定时器，等待目的端确认收到这个报文段。如果超过某个时间还没有收到确认，将重发这个报文段。

#### 停止 - 等待协议

在已发送但未确认的报文被确认之前，发送方的滑动窗口将不会滑动（类比滑动窗口算法）

#### 最大连接数限制

- **Client 最大 TCP 连接数**：TCP 端口的数据类型是 unsigned short（$2^{16}$），可用端口最多有 65535 个（除端口 0）
- **Server 最大 TCP 连接数**：客户端 IP 数 × 客户端 port 数（理论）

> 对 IPV4，在不考虑 IP 地址分类的情况下，最大 TCP 连接数约为 $2^{32}$（IP 数）× $2^{16}$（port 数），即约为 $2^{48}$

### 流量控制

所谓流量控制就是让发送方的发送速率不要太快，让接收方来得及接收。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112212120334.png" style="zoom: 33%;" />

### 拥塞控制

拥塞控制是作用于网络的，它是防止过多的数据注入到网络中，避免出现网络负载过大的情况。

常用的解决方法有：慢开始和拥塞避免、快重传和快恢复。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112212127381.png" style="zoom: 40%;" />

> **拥塞控制和流量控制的区别：**
> 拥塞控制往往是一种全局的控制，防止过多的数据注入到网络之中
> 流量控制往往指点对点通信量的控制，是端到端的问题（TCP 连接的端点只要不能收到对方的确认信息，猜想在网络中发生了拥塞，但并不知道发生在何处）

## UDP

### 报文

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112212131402.png" style="zoom: 33%;" />

### 区别

<table>
<thead>
<tr>
<th>类型</th>
<th>是否面向连接</th>
<th>传输可靠性</th>
<th>传输形式</th>
<th>传输效率</th>
<th>所需资源</th>
<th>应用场景</th>
<th>首部字节</th>
</tr>
</thead>
<tbody>
<tr>
<td>TCP</td>
<td>是</td>
<td>可靠</td>
<td>字节流</td>
<td>慢</td>
<td>多</td>
<td>文件传输、邮件传输</td>
<td>20~60</td>
</tr>
<tr>
<td>UDP</td>
<td>否</td>
<td>不可靠</td>
<td>数据报文段</td>
<td>快</td>
<td>少</td>
<td>即时通讯、域名转换</td>
<td>8 个字节</td>
</tr>
</tbody>
</table>

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112212058636.png" style="zoom: 50%;" />

### 不可靠传输

- UDP 只有一个 socket 接收缓冲区，没有 socket 发送缓冲区，即只要有数据就发，当对方接收缓冲区满了后就会丢弃，因此 UDP 不能保证数据能够到达目的地
- UDP 没有流量控制和重传机制
- UDP 中调用 connect 只是把对端的 IP 和 端口号记录下来，并且 UDP 可多次调用 connect 来指定一个新的 IP 和端口号，或者断开旧的 IP 和端口号

> 调用 connect 的 UDP 会提升效率，并且在高并发服务中会增加系统稳定性。
> 调用 bind 函数时，就会将这个套接字指定一个端口，若不调用 bind 函数，系统内核会随机分配一个端口给该套接字。
> 当手动绑定时，能够避免内核来执行这一操作，从而在一定程度上提高性能。

## More

### SYN FLOOD

SYN Flood 是种典型的 DoS（拒绝服务）攻击，其目的是通过消耗服务器所有可用资源使服务器无法用于处理合法请求。

通过重复发送初始连接请求（SYN）数据包，攻击者能够压倒目标服务器上的所有可用端口，导致目标设备根本不响应合法请求。
