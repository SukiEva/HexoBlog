---
title: Jetpack ViewModel
date: 2021-09-30 08:58
tags:
  - Jetpack
categories: [Framework,Android]
toc: true
updated: 2021-09-30 08:58
references:
  - title: ViewModel 概览  | Android 开发者  | Android Developers
    url: https://developer.android.com/topic/libraries/architecture/viewmodel
  - title: 《第一行代码 Android 第3版》
    url: https://guolin.blog.csdn.net/article/details/105233078
---

`ViewModel` 类旨在以注重生命周期的方式存储和管理界面相关的数据。`ViewModel` 类让数据可在发生屏幕旋转等配置更改后继续留存。

ViewModel 具有生命周期意识，会自动存储和管理 UI 相关的数据，即使设备配置发生变化后数据还会存在，我们就不需要在 onSaveInstanceState 保存数据，在 onCreate 中恢复数据了，使用 ViewModel 这部分工作就不需要我们做了，很好地将视图与逻辑分离开来。

<!-- more -->

## 生命周期

`ViewModel` 的一个重要作用是帮助 `Activity` 分担部分工作，专门用来存放与界面相关的数据，减轻 `Activity` 负担。

另外，如果系统销毁或重新创建界面控制器，则存储在其中的任何瞬态界面相关数据都会丢失。而 `ViewModel` 的生命周期和 `Activity` 不同，只有 `Activity` 退出时才会跟着一起销毁。

`ViewModel` 对象存在的时间范围是获取 `ViewModel` 时传递给 [`ViewModelProvider`](https://developer.android.com/reference/androidx/lifecycle/ViewModelProvider) 的 [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle)。`ViewModel` 将一直留在内存中，直到限定其存在时间范围的 `Lifecycle` 永久消失：

- 对于 activity，是在 activity 完成时；
- 对于 fragment，是在 fragment 分离时。

> 通常在系统首次调用 Activity 对象的 `onCreate()` 方法时请求 `ViewModel`

<center><img src="https://developer.android.com/images/topic/libraries/architecture/viewmodel-lifecycle.png" alt="ViewModel 的生命周期 " style="zoom:80%;" /></center>

> todo
