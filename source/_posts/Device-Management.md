---
title: 设备管理
tags:
  - Device
categories: [Computer Basics,Operating System]
toc: true
date: 2022-01-21 14:27
updated: 2022-01-21 14:27
references:
  - title: 八股文（星球精华汇总）
    url: https://wx.zsxq.com/dweb2/index/topic_detail/185515824245112
  - title: 设备管理 - leetcode
    url: https://leetcode-cn.com/leetbook/read/tech-interview-cookbook/or7mr5/
---

磁盘是计算机主要的存储介质，可以存储大量的二进制数据，并且断电后也能保持数据不丢失。

早期计算机使用的磁盘是软磁盘（Floppy Disk，简称软盘），如今常用的磁盘是硬磁盘（Hard disk，简称硬盘）。

电脑外设就是除主机外的大部分硬件设备都可称作外部设备，或叫外围设备，简称外设。
计算机系统没有输入输出设备，就如计算机系统没有软件一样，是毫无意义的。

<!-- more -->

## 磁盘

### 结构

- 盘面（Platter）：一个磁盘有多个盘面；
- 磁道（Track）：盘面上的圆形带状区域，一个盘面可以有多个磁道；
- 扇区（Track Sector）：磁道上的一个弧段，一个磁道可以有多个扇区，它是最小的物理储存单位，目前主要有 512 bytes 与 4 K 两种大小；
- 磁头（Head）：与盘面非常接近，能够将盘面上的磁场转换为电信号（读），或者将电信号转换为盘面的磁场（写）；
- 制动手臂（Actuator arm）：用于在磁道之间移动磁头；
- 主轴（Spindle）：使整个盘面转动。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202201211431280.png"  />

### 磁盘调度算法

读写一个磁盘块的时间的影响因素有：

- 旋转时间（主轴转动盘面，使得磁头移动到适当的扇区上）
- 寻道时间（制动手臂移动，使得磁头移动到适当的磁道上）
- 实际的数据传输时间

其中，寻道时间最长，因此磁盘调度的主要目标是使磁盘的平均寻道时间最短。

#### 先来先服务

> FCFS, First Come First Served

按照磁盘请求的顺序进行调度。

优点是公平和简单。

缺点也很明显，因为未对寻道做任何优化，使平均寻道时间可能较长。

#### 最短寻道时间优先

> SSTF, Shortest Seek Time First

优先调度与当前磁头所在磁道距离最近的磁道。

虽然平均寻道时间比较低，但是不够公平。

如果新到达的磁道请求总是比一个在等待的磁道请求近，那么在等待的磁道请求会一直等待下去，也就是出现饥饿现象。

具体来说，两端的磁道请求更容易出现饥饿现象。

<img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/4e2485e4-34bd-4967-9f02-0c093b797aaa.png" alt="image" style="zoom:80%;" />

#### 电梯算法

> SCAN

电梯总是保持一个方向运行，直到该方向没有请求为止，然后改变运行方向。

电梯算法（扫描算法）和电梯的运行过程类似，总是按一个方向来进行磁盘调度，直到该方向上没有未完成的磁盘请求，然后改变方向。

因为考虑了移动方向，因此所有的磁盘请求都会被满足，解决了 SSTF 的饥饿问题。

<img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/271ce08f-c124-475f-b490-be44fedc6d2e.png" alt="image" style="zoom:80%;" />

## 外设

### 如何让外设动起来

1. CPU 向外设的控制器发送指令，即 out 指令
2. 形成 ⽂件视图（为了统⼀ out 指令的形式）
3. 中断（外设处理完事后，需要通知 cpu 继续接手下⼀步处理）

### 显示器如何工作

> Printf 函数的工作流程

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202201211437690.png" alt="image-20220121143755635" style="zoom: 67%;" />

### 键盘如何工作

1. 中断处理（根据扫描码获取 对应的 ascii 码）
2. 将对应的 ascii 码加入缓冲队列 read_que 中，等待上层程序调用
