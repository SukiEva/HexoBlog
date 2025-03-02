---
title: Jetpack Lifecycle
date: 2021-10-01 19:37
tags:
  - Jetpack
categories: [Framework,Android]
toc: true
updated: 2021-10-01 19:37
references:
  - title: Lifecycle  | Android 开发者  | Android Developers
    url: https://developer.android.com/jetpack/androidx/releases/lifecycle
  - title: Android官方架构组件:Lifecycle详解&原理分析_却把清梅嗅的博客-CSDN博客_lifecycle原理
    url: https://qingmei2.blog.csdn.net/article/details/79029657
---

`Lifecycle` 组件可以让任何一个类都能轻松感知到 `Activity` 的生命周期，同时不需要再 `Activity` 中编写太多额外的逻辑。

<!-- more -->

## 生命周期

生命周期和状态事件如图所示：

<center>
<img src="https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.svg?hl=zh-cn" alt=" 构成 Android Activity 生命周期的状态和事件 " width="65%" />
</center>

## Lifecycles 的最佳实践

一般在 `ViewModel` 中观察和控制界面。

- 保持 UI 控制器（Activity 和 Fragment）尽可能的精简。它们不应该试图去获取它们所需的数据；相反，要用 ViewModel 来获取，并且观察 LiveData 将数据变化反映到视图中。
- 尝试编写数据驱动（data-driven）的 UI，即 UI 控制器的责任是在数据改变时更新视图或者将用户的操作通知给 ViewModel。
- 将数据逻辑放到 ViewModel 类中。ViewModel 应该作为 UI 控制器和应用程序其它部分的连接服务。注意：不是由 ViewModel 负责获取数据（例如：从网络获取）。相反，ViewModel 调用相应的组件获取数据，然后将数据获取结果提供给 UI 控制器。
- 使用 Data Binding 来保持视图和 UI 控制器之间的接口干净。这样可以让视图更具声明性，并且尽可能减少在 Activity 和 Fragment 中编写更新代码。如果你喜欢在 Java 中执行该操作，请使用像 Butter Knife 这样的库来避免使用样板代码并进行更好的抽象化。
- 如果 UI 很复杂，可以考虑创建一个 Presenter 类来处理 UI 的修改。虽然通常这样做不是必要的，但可能会让 UI 更容易测试。
- 不要在 ViewModel 中引用 View 或者 Activity 的 context。因为如果 ViewModel 存活的比 Activity 时间长（在配置更改的情况下），Activity 将会被泄漏并且无法被正确的回收。
