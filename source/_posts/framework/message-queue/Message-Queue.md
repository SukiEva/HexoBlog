---
title: 消息队列
toc: true
categories: [Framework,Message Queue]
tags: Message Queue
date: 2022-10-15 21:27
updated: 2022-10-15 22:04
references:
	- title: Java Guide
	  url: https://javaguide.cn/high-performance/message-queue/message-queue.html
	- title: 消息队列 - 维基百科
	  url: https://zh.wikipedia.org/wiki/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97
	- title: Advanced Java
	  url: https://doocs.github.io/advanced-java/#/docs/high-concurrency/why-mq
---

在操作系统中，**消息队列**（Message queue）是一种进程间通信或同一进程的不同线程间的通信方式。

消息队列可以看作是一个存放消息的容器，是一种**异步**的通信方式。

<!-- more -->

## 为什么使用消息队列

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202210152143172.awebp" alt=" 消息队列 " style="zoom:67%;" />

比较核心的有 3 个：**异步**、**削峰**、**解耦**。

### 异步

通过异步处理，能够提高系统性能（减少响应所需时间）。

> 一般互联网类的企业，对于用户直接的操作，一般要求是每个请求都必须在 200 ms 以内完成，对用户几乎是无感知的。

### 削峰/限流

先将短时间高并发产生的事务消息存储在消息队列中，然后后端服务再慢慢根据自己的能力去消费这些消息，这样就避免直接把后端服务打垮掉。

> 电子商务一些秒杀、促销活动中，合理使用消息队列可以有效抵御促销活动刚开始大量订单涌入对系统的冲击。

### 解耦

消息队列使用发布 - 订阅模式工作，消息发送者（生产者）发布消息，一个或多个消息接受者（消费者）订阅消息。

对新增业务，只要对该类消息感兴趣，即可订阅该消息，对原有系统和业务没有任何影响，从而实现网站业务的可扩展性设计。

## 消息队列的缺点

- **系统可用性降低：** 需要考虑消息丢失或者 MQ 挂掉等情况。
- **系统复杂性提高：** 需要保证消息没有被重复消费、处理消息丢失的情况、保证消息传递的顺序性等问题。
- **一致性问题：** 万一消息的真正消费者并没有正确消费消息，就会导致数据不一致的情况。

## 常见的消息队列

<table>
<thead>
<tr>
<th>特性</th>
<th>ActiveMQ</th>
<th>RabbitMQ</th>
<th>RocketMQ</th>
<th>Kafka</th>
</tr>
</thead>
<tbody><tr>
<td>单机吞吐量</td>
<td>万级，比 RocketMQ、Kafka 低一个数量级</td>
<td>同 ActiveMQ</td>
<td>10 万级，支撑高吞吐</td>
<td>10 万级，高吞吐，一般配合大数据类的系统来进行实时数据计算、日志采集等场景</td>
</tr>
<tr>
<td>topic 数量对吞吐量的影响</td>
<td></td>
<td></td>
<td>topic 可以达到几百/几千的级别，吞吐量会有较小幅度的下降，这是 RocketMQ 的一大优势，在同等机器下，可以支撑大量的 topic</td>
<td>topic 从几十到几百个时候，吞吐量会大幅度下降，在同等机器下，Kafka 尽量保证 topic 数量不要过多，如果要支撑大规模的 topic，需要增加更多的机器资源</td>
</tr>
<tr>
<td>时效性</td>
<td>ms 级</td>
<td>微秒级，这是 RabbitMQ 的一大特点，延迟最低</td>
<td>ms 级</td>
<td>延迟在 ms 级以内</td>
</tr>
<tr>
<td>可用性</td>
<td>高，基于主从架构实现高可用</td>
<td>同 ActiveMQ</td>
<td>非常高，分布式架构</td>
<td>非常高，分布式，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用</td>
</tr>
<tr>
<td>消息可靠性</td>
<td>有较低的概率丢失数据</td>
<td>基本不丢</td>
<td>经过参数优化配置，可以做到 0 丢失</td>
<td>同 RocketMQ</td>
</tr>
<tr>
<td>功能支持</td>
<td>MQ 领域的功能极其完备</td>
<td>基于 erlang 开发，并发能力很强，性能极好，延时很低</td>
<td>MQ 功能较为完善，还是分布式的，扩展性好</td>
<td>功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用</td>
</tr>
</tbody></table>

> **中小型公司**，技术实力较为一般，技术挑战不是特别高，用 RabbitMQ 是不错的选择；
>
> **大型公司**，基础架构研发实力较强，用 RocketMQ 是很好的选择；
>
> 如果是**大数据领域**的实时计算、日志采集等场景，用 Kafka 是业内标准的。
