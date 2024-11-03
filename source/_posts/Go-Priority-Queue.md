---
title: Go中的优先队列
date: 2021-11-01 22:13
tags:
  - Queue
categories: [Computer Basics,Data Structure]
toc: true
updated: 2021-11-01 22:13
references:
  - title: Go标准库中文文档 (cngolib.com)
    url: http://cngolib.com/container-heap.html
---

* 论如何在 Go 语言中使用优先队列。*

<!-- more -->

## 介绍

Go 提供了 [container/heap](https%3A%2F%2Fgolang.org%2Fpkg%2Fcontainer%2Fheap%2F) 这个包来实现堆的操作。堆实际上是一个树的结构，每个元素的值都是它的子树中最小的，因此根节点 `index = 0` 的值是最小的，即最小堆。

堆也是实现优先队列 Priority Queue 的常用方式。

堆中元素的类型需要实现 `heap.Interface` 这个接口：

```go
type Interface interface {
    sort.Interface
    Push(x interface{}) // add x as element Len()
    Pop() interface{}   // remove and return element Len() - 1.
}
```

其中 [sort.Interface](https%3A%2F%2Fgolang.org%2Fpkg%2Fsort%2F%23Interface) 包括 `Len()`, `Less`, `Swap` 方法。

## 实现

```go
type IntHeap [][2]int // 0 key 1 value

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i][1] < h[j][1] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.([2]int))
}

func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}


// 347. 前 K 个高频元素
// https://leetcode-cn.com/problems/top-k-frequent-elements/
func topKFrequent(nums []int, k int) []int {
	m := make(map[int]int)
	ans := make([]int, k)
	h := &IntHeap{}
	heap.Init(h)
	for _, v := range nums {
		m[v]++
	}
	for key, value := range m {
		heap.Push(h, [2]int{key, value})
		if h.Len() > k {
			heap.Pop(h)
		}
	}
	for k > 0 {
		k--
		ans[k] = heap.Pop(h).([2]int)[0]
	}
	return ans
}
```
