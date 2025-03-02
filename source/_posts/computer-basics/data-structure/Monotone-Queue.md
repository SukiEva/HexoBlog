---
title: 单调队列 Monotone Queue
date: 2021-11-01 21:19
tags:
  - Queue
categories: [Computer Basics,Data Structure]
toc: true
updated: 2021-11-01 21:19
references:
  - title: 代码随想录 (programmercarl.com)
    url: https://programmercarl.com/0239.滑动窗口最大值.html
---

***“如果一个选手比你小还比你强，你就可以退役了。”**——单调队列的原理 *

<!-- more -->

## 介绍

单调队列的基本思想是，维护一个双向队列（deque），遍历序列，仅当一个元素**可能**成为某个区间最值时才保留它。

![单调队列如何维护队列里的元素](https://code-thinking.cdn.bcebos.com/gifs/239.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E6%9C%80%E5%A4%A7%E5%80%BC.gif)

设计单调队列的时候，pop，和 push 操作要保持如下规则：

1. pop(value)：如果窗口移除的元素 value 等于单调队列的出口元素，那么队列弹出元素，否则不用任何操作
2. push(value)：如果 push 的元素 value 大于入口元素的数值，那么就将队列入口的元素弹出，直到 push 元素的数值小于等于队列入口元素的数值为止

保持如上规则，每次窗口移动的时候，只要问 que.front() 就可以返回当前窗口的最大值。

以题目示例为例，输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3，动画如下：

![239.滑动窗口最大值-2](https://code-thinking.cdn.bcebos.com/gifs/239.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E6%9C%80%E5%A4%A7%E5%80%BC-2.gif)

## 实现

> 保证队列里单调递减或递增的原则，所以叫做单调队列。

以下不是固定写法：

```go
type MonotoneQueue struct {
	queue []int
}

func Constructor() MonotoneQueue {
	return MonotoneQueue{
		queue: make([]int, 0),
	}
}

func (this *MonotoneQueue) Front() int {
	return this.queue[0]
}

func (this *MonotoneQueue) Back() int {
	return this.queue[len(this.queue)-1]
}

func (this *MonotoneQueue) Empty() bool {
	return len(this.queue) == 0
}

func (this *MonotoneQueue) Push(x int) {
	for !this.Empty() && x > this.Back() {
		this.queue = this.queue[:len(this.queue)-1]
	}
	this.queue = append(this.queue, x)
}

func (this *MonotoneQueue) Pop(x int) {
	if !this.Empty() && this.Front() == x {
		this.queue = this.queue[1:]
	}
}
```
