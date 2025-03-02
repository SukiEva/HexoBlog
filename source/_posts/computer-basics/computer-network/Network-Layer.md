---
title: 网络层
toc: true
date: 2021-12-22 14:57
tags:
  - Network
categories: [Computer Basics,Computer Network]
updated: 2021-12-22 14:57
references:
  - title: LeetCode
    url: https://leetcode-cn.com/leetbook/read/networks-interview-highlights/esz3b2/
  - title: 王道考研
    url: http://cskaoyan.com/forum.php
---

**网络层**（**Network Layer**）是 OSI 模型 中的第三层（TCP/IP 模型中的网际层），提供路由和寻址的功能，使两终端系统能够互连且决定最佳路径，并具有一定的拥塞控制和流量控制的能力。
由于 TCP/IP 协议体系中的网络层功能由 IP 协议规定和实现，故又称 IP 层。

网络层协议负责提供**主机**间的逻辑通信；传输层协议负责提供**进程**间的逻辑通信。

功能：

- 路由选择与分组转发 **最佳路径**
- 异构网络互联
- 拥塞控制

网络层重要协议：

- IP（Internet Protocol）网际互连协议

与 IP 协议配套使用的还有三个协议：

- ARP（Address Resolution Protocol）地址解析协议
- ICMP（Internet Control Message Protocol）网际控制报文协议
- IGMP（Internet Group Management Protocol）网际组管理协议

<!-- more -->

## IP

该协议工作在网络层，主要目的就是为了提高网络的可扩展性。

和传输层 TCP 相比，IP 协议提供一种无连接/不可靠、尽力而为的数据包传输服务，其与 TCP 协议（传输控制协议）一起构成了 TCP/IP 协议族的核心。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221614969.png" style="zoom: 67%;" />

IP 协议主要有以下几个作用：

- **寻址和路由**：在 IP 数据包中会携带源 IP 地址和目的 IP 地址来标识该数据包的源主机和目的主机。IP 数据报在传输过程中，每个中间节点（IP 网关、路由器）只根据网络地址进行转发，如果中间节点是路由器，则路由器会根据路由表选择合适的路径。IP 协议根据路由选择协议提供的路由信息对 IP 数据报进行转发，直至抵达目的主机。
- **分段与重组**：IP 数据包在传输过程中可能会经过不同的网络，在不同的网络中数据包的最大长度限制是不同的，IP 协议通过给每个 IP 数据包分配一个标识符以及分段与组装的相关信息，使得数据包在不同的网络中能够传输，被分段后的 IP 数据报可以独立地在网络中进行转发，在到达目的主机后由目的主机完成重组工作，恢复出原来的 IP 数据包。

### 数据报

总长度单位 1B，片位移单位 8B，首部单位 4B

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221622589.png" style="zoom:50%;" />

> **生存时间** ：TTL，它的存在是为了防止无法交付的数据报在互联网中不断兜圈子。以路由器跳数为单位，当 TTL 为 0 时就丢弃数据报。

#### 分片

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221624491.png" style="zoom: 60%;" />

### IPV4

#### IP 地址分类

IP 地址由两部分组成，网络号和主机号，其中不同分类具有不同的网络号长度，并且是固定的。

`IP 地址 ::= {< 网络号 >, < 主机号 >}`

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221637019.png" style="zoom: 33%;" />

以上分类还空出一些地址，这些特殊的地址包括：

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221639252.png" style="zoom:60%;" />

另外还有私有地址（局域网）：

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221641441.png" style="zoom: 33%;" />

#### 子网划分

通过在主机号字段中拿一部分作为子网号，把两级 IP 地址划分为三级 IP 地址。

`IP 地址 ::= {< 网络号 >, < 子网号 >, < 主机号 >}`

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221643433.png" style="zoom: 40%;" />

子网掩码：主机位全 0，其他全 1（网络位、子网位）

#### NAT 网络地址转换

NAT（Network Address Translation)，它是一种把内部私有网络地址翻译成公有网络 IP 地址的技术。

该技术不仅能解决 IP 地址不足的问题，而且还能隐藏和保护网络内部主机，从而避免来自外部网络的攻击。

> 在专用网连接到因特网（公用地址）的路由器上安装 NAT 软件（NAT 路由器），它至少有一个有效的外部全球 IP 地址
>
> - NAT 路由器根据转换表替换源 IP 地址或者目的 IP 地址和端口号，所以转发数据报时需查看和转换传输层的端口号
> - 普通路由器仅工作在网络层，不改变源 IP 和目的 IP

NAT 的实现有三种方式：

- 静态转换，一对一，一个私有对应一个公有
- 动态转换，一对多，一个私有每次转换的公有不唯一
- 端口多路复用，多对一，多个私有共享一个合法的外部 IP，映射到了不同的端口上

#### CIDR 无分类域间路由选择

CIDR（Classless Inter-Domain Routing）

无分类编址 CIDR 消除了传统 A 类、B 类和 C 类地址以及划分子网的概念，使用网络前缀和主机号来对 IP 地址进行编码，网络前缀的长度可以根据需要变化。

`IP 地址 ::= {< 网络前缀号 >, < 主机号 >}`

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221647430.png" style="zoom: 40%;" />

### IPV6

从根本上解决 IPV4 地址不够的问题。

> 其他方法：
> NAT：网络地址转换
> DHCP：动态主机配置协议。动态分配 IP 地址，只给接入网络的设备分配 IP 地址
> （DHCP 是应用层协议，使用客户/服务器方式，客户端和服务端通过广播方式进行交互，基于 UDP）

IPV6 将地址从 32 位（4B）扩大到了 128 位（64B）

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221715771.png" style="zoom: 78%;" />

## ARP

ARP（Address Resolution Protocol）地址解析协议，完成 IP 地址到 MAC 地址的映射（解决下一跳走哪的问题）

### 使用过程

1. 检查 ARP 高速缓存，有对应表项则写入 MAC 帧，没有则用目的 MAC 地址为 FF-FF-FF-FF-FF-FF 的帧封装并**广播 ARP 请求分组**，同一局域网内所有主机都能受到该请求
2. 目的主机收到请求后就会向源主机单播一个 ARP 响应分组，源主机收到后将此映射写入 ARP 缓存（10～20min 更新一次）

### 典型情况

- 主机 A 发给本网络上的主机 B：用 ARP 找到 B 的硬件地址
- 主机 A 发给另一网络上的主机 B：用 ARP 找到本网络上一个路由器（网关）的硬件地址
- 路由器发给本网络的主机 B：用 ARP 找到 B 的硬件地址
- 路由器发给另一网络的主机 B：用 ARP 找到本网络上的一个路由器的硬件地址

## ICMP

ICMP（Internet Control Message Protocol）是网际控制报文协议，主要是实现 IP 协议中未实现的部分功能，是一种网络层协议。该协议并不传输数据，只传输控制信息来辅助网络层通信。

### 功能

- 验证网络是否畅通（确认接收方是否成功接收到 IP 数据包）
- 辅助 IP 协议实现可靠传输（若发生 IP 丢包，ICMP 会通知发送方 IP 数据包被丢弃的原因，之后发送方会进行相应的处理）

### 应用

- **Ping**：测试两个主机间的连通性，使用 ICMP 会送请求和回答报文
- **TraceRoute**：跟踪一个分组从源点到终点的路径，使用 ICMP 时间超过差错报告报文

> ping 不通可能存在的问题：
>
> - 首先看网络是否连接正常，检查网卡驱动是否正确安装
> - 局域网设置问题，检查 IP 地址是否设置正确
> - 看是否被防火墙阻拦（有些设置中防火墙会对 ICMP 报文进行过滤），如果是的话，尝试关闭防火墙
> - 看是否被第三方软件拦截
> - 两台设备间的网络延迟是否过大（例如路由设置不合理），导致 ICMP 报文无法在规定的时间内收到

 IGMP

IGMP（Internet Group Management Protocol）网际组管理协议，让路由器知道局域网上是否有主机（的进程）参加或退出某个组播组。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221718842.png" style="zoom: 67%;" />

 路由器

路由器是网络层设备，通过数据包中的目的 IP 地址识别不同的网络从而确定数据转发的目的地址，网络号是唯一的。

路由器根据路由选择协议和路由表信息从而确定数据的转发路径，直到到达目的网络，它工作于网络层。

> 交换机用于局域网，利用主机的物理地址（MAC 地址）确定数据转发的目的地址，它工作于数据链路层。

## 分组转发流程

1. 从 IP 数据包中提取出目的主机的 IP 地址，找到其所在的网络
2. 判断目的 IP 地址所在的网络是否与本路由器直接相连，如果是，则不需要经过其它路由器**直接交付**，否则执行 3
3. 检查路由表中是否有目的 IP 地址的特定主机路由。如果有，则按照路由表传送到下一跳路由器中，否则执行 4
4. 逐条检查路由表，使用每一行的子网掩码与目的 IP 匹配。若找到匹配路由，则按照路由表转发到下一跳路由器中，否则执行步骤 5
5. 若路由表中设置有默认路由，则按照默认路由转发到默认路由器中，否则执行步骤 6
6. 无法找到合适路由，向源主机报错

## 路由协议

- RIP（Routing Information Protocol）路由信息协议
- OSPF（Open Shortest Path First）开放式最短路径优先
- BGP（Border Gateway Protocol）边界网关协议

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112221732310.png" style="zoom:70%;" />
