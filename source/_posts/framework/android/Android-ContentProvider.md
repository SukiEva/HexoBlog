---
title: Android四大组件之ContentProvider
date: 2021-09-28 22:05
tags:
  - Component
categories: [Framework,Android]
toc: true
updated: 2021-09-28 22:05
references:
  - title: 内容提供程序  | Android 开发者  | Android Developers
    url: https://developer.android.com/guide/topics/providers/content-providers?hl=zh-cn
  - title: 《第一行代码 Android 第3版》
    url: https://guolin.blog.csdn.net/article/details/105233078
---

内容提供程序主要目的是不同应用程序间的数据共享。

它提供了一套完整的机制，允许一个程序访问另一个程序的数据，同时保证被访问数据的安全性。

目前，使用 ContentProvider 是 Android 实现跨程序共享数据的标准方式。

<!-- more -->

> todo
