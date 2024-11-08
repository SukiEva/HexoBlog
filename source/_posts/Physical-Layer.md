---
title: 物理层
tags:
  - Network
categories: [Computer Basics,Computer Network]
toc: true
date: 2021-12-23 21:47
updated: 2021-12-23 21:47
---

**物理层**（Physical Layer）是计算机网络 OSI 模型中最低的一层，也是最基本的一层。简单的说，网络的物理层面确保原始的数据可在各种物理媒体上传输。

物理层解决如何在连接各种计算机的传输媒体上**传输数据比特流**

物理层主要任务：确定与传输媒体接口有关的一些特性

四大特性：

- 机械特性：定义物理连接的特性，规定物理连接时所采用的规格、接口形状、引线数目、引脚数量和排列情况
- 电气特性：规定传输二进制位时，线路上信号的电压范围、阻抗匹配、传输速率和距离限制
- 功能特性：指明某条线上出现的某一电平表示何种意义，接口部件的信号线的用途
- 规程特性：定义各条物理线路的工作规程和时序关系

物理层设备：中继器

<!-- more -->

## 通信方式

![](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112232153357.png)

主要考虑：

- 单工通信：单向传输
- 半双工通信：双向交替传输
- 全双工通信：双向同时传输

## 数据交换方式

- 电路交换
- 报文交换
- 分组交换

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202112232156819.png" style="zoom: 40%;" />

## 传输介质

### 导向性传输介质

- 双绞线
- 同轴电缆
- 光纤

### 非导向性传输介质

- 无线电波
- 微波
- 红外线、激光
