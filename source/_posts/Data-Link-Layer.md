---
title: 数据链路层
tags:
  - Network
categories: [Computer Basics,Computer Network]
toc: true
date: 2021-12-23 20:57
updated: 2021-12-23 20:57
references:
  - title: 王道考研
    url: http://cskaoyan.com/forum.php
---

**数据链路层** 是 OSI 参考模型中的第二层，介乎于物理层和网络层之间。

功能：在物理层提供服务的基础上向网络层提供服务

- 最基本的服务：**将源自于物理层的数据可靠地传输到相邻结点到目标机网络层**
- 主要作用：加强物理层传输原始比特流的功能，将物理层提供的可能出错的物理连接改造成逻辑上无差错的数据链路，使之对网络层表现为一条无差错的链路
- 为网络层提供服务：
	- 无确认的无连接服务
	- 有确认的无连接服务
	- 有确认的面向连接服务

重要协议：

- PPP（Point to Point Protocol）点 - 点协议

链路层设备：

- 交换机
- 网桥

<!-- more -->

## 基本问题

### 封装成帧

将网络层传下来的分组添加首部和尾部，用于标记帧的开始和结束。

### 透明传输

透明表示一个实际存在的事物看起来好像不存在一样。
针对用户是透明的；首尾是界定帧，转义字符去除数据部分和首尾相同引起歧义。

### 差错控制

链路层编码针对**一组比特**，通过冗余码的技术实现一组二进制比特串在传输过程中是否出现差错。

目前数据链路层广泛使用了循环冗余检验（CRC）来检查比特差错。

主要包括：

- 检错编码
	- 奇偶校验码
	- 循环冗余码
- 纠错编码
	- 海明码

## 流量控制

主要通过滑动窗口协议，根据窗口大小分为：

- 停止 - 等待协议：发送窗口 = 1，接收窗口 = 1
- 后退 N 帧协议（GBN）：发送窗口 > 1，接收窗口 = 1
- 选择重传协议（SR）：发送窗口 > 1，接收窗口 > 1

<table>
<tr>
	<td><img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112232122980.png" /></td>
	<td><img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112232123547.png" /></td>
	<td><img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112232123892.png" /></td>
</tr>
</table>

## 介质访问控制

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112232127804.png" style="zoom: 33%;" />

### CSMA/CD

> CSMA/CD 为载波侦听多路访问/冲突检测，是像以太网这种广播网络采用的一种机制，我们知道在以太网中多台主机在同一个信道中进行数据传输，CSMA/CD 很好的解决了共享信道通信中出现的问题，它的工作原理主要包括两个部分：
>
> - **载波监听**：当使用 CSMA/CD 协议时，总线上的各个节点都在监听信道上是否有信号在传输，如果有的话，表明信道处于忙碌状态，继续保持监听，直到信道空闲为止。如果发现信道是空闲的，就立即发送数据。
> - **冲突检测**：当两个或两个以上节点同时监听到信道空闲，便开始发送数据，此时就会发生碰撞（数据的传输延迟也可能引发碰撞）。当两个帧发生冲突时，数据帧就会破坏而失去了继续传输的意义。在数据的发送过程中，以太网是一直在监听信道的，当检测到当前信道冲突，就立即停止这次传输，避免造成网络资源浪费，同时向信道发送一个「冲突」信号，确保其它节点也发现该冲突。之后采用一种二进制退避策略让待发送数据的节点随机退避一段时间之后重新。

CSMA/CD 的工作流程可以概括为：

1. 先听后发
2. 边发边听
3. 冲突停发
4. 随机重发

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112232130101.png" style="zoom:33%;" />

> CSMA/CD: 载波监听多路访问 / 碰撞检测（Detect），用于有线局域网（LAN）
> CSMA/CA: 载波监听多路访问 / 碰撞避免（Avoid），用于无线局域网（WIFI）

## PPP 协议

互联网用户通常需要连接到某个 ISP 之后才能接入到互联网，PPP（点对点） 协议是用户计算机和 ISP 进行通信时所使用的数据链路层协议。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112232140255.png" style="zoom: 33%;" />

## MAC 地址

MAC 地址是链路层地址，长度为 6 字节（48 位），用于唯一标识网络适配器（网卡）。

一台主机拥有多少个网络适配器就有多少个 MAC 地址。

> MAC 地址是数据链路层和物理层使用的地址，是写在网卡上的物理地址。MAC 地址用来定义网络设备的位置。
> IP 地址是网络层和以上各层使用的地址，是一种逻辑地址。IP 地址用来区别网络上的计算机。

### 为什么需要 MAC 地址

只有当设备连入网络时，才能根据他进入了哪个子网来为其分配 IP 地址，在设备还没有 IP 地址的时候或者在分配 IP 地址的过程中，我们需要 MAC 地址来区分不同的设备。

### 为什么需要 IP 地址

光有 MAC 地址的话，寻址困难。IP 地址和地域有关，可以分区域寻址，效率更高。
