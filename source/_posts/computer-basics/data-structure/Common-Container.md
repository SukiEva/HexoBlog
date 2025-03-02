---
title: 常见容器底层
date: 2021-11-02 09:10
tags:
  - Container
categories: [Computer Basics,Data Structure]
toc: true
updated: 2021-11-02 09:10
references:
  - title: C++ STL容器底层数据结构总结 - 简书 (jianshu.com)
    url: https://www.jianshu.com/p/834cc223bb57
  - title: C++面试进阶（STL底层数据结构特点及实现） - 知乎 (zhihu.com)
    url: https://zhuanlan.zhihu.com/p/359878588
plugins:
  - mathjax
---

各种语言下，常见容器底层整理。

<!-- more -->

## C++

### Vector

其底层数据结构是**数组**，由于能动态扩容，所以也称**动态数组**

**特点：**

- 随机访问：$O(1)$
- 随机插入删除： $O(n)$，中间插入会引起后面数据的拷贝，尾部可快速增删
- 扩容规则：
	- 新建时初始化一片空间
	- 插入元素引起扩容的时候，gcc 会申请 2 倍的空间，拷贝原有数据
	- 释放原来空间
- 在进行迭代器相关的修改操作时（包括扩容），**所有迭代器和指针引用都会失效**

### Map & Multimap & Set & Multiset

map 提供一对一 key-value 的数据处理能力；set 可以理解为关键字即值，即只保存关键字的容器。

与 multi 的区别在于，multi 允许关键字重复，而上面 2 个不允许重复。

底层数据结构均为**红黑树**，可以参考 [教你透彻了解红黑树]([The-Art-Of-Programming-By-July/03.01.md at master · julycoding/The-Art-Of-Programming-By-July (github.com)](https://github.com/julycoding/The-Art-Of-Programming-By-July/blob/master/ebook/zh/03.01.md))。

**特点：**

- 访问、查找、删除：$O(logn)$

### unordered_map & unordered_multimap & unordered_set & unordered_mutiset

顾名思义，以上容器是无序的，所以底层实现为**哈希表**，因此其查找时间复杂度理论上达到了**O(n)**

**特点：**

- 访问、查找、删除：$O(1)$，但是实际上要考虑碰撞的问题

## Go

Go 语言不添加其他包的话，只提供了一个 map 容器，不过其他容器适配器之类可以用万能的数组切片实现

### Map

底层数据结构是哈希表

**特点：**

- 访问、查找、删除：$O(1)$
