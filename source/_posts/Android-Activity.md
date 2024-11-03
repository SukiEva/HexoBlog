---
title: Android四大组件之Activity
date: 2021-09-28 08:45
tags:
  - Component
categories: Android
toc: true
updated: 2021-09-28 08:45
references:
  - title: Activity 简介  | Android 开发者  | Android Developers
    url: https://developer.android.com/guide/components/activities/intro-activities?hl=zh-cn
  - title: 《第一行代码 Android 第3版》
    url: https://guolin.blog.csdn.net/article/details/105233078
---

`Activity` 类是 Android 应用的关键组件，而 Activity 的启动和组合方式则是该平台应用模型的基本组成部分。

在编程范式中，应用是通过 `main()` 方法启动的，而 Android 系统与此不同，它会调用与其生命周期特定阶段相对应的特定回调方法来启动 `Activity` 实例中的代码。

<!-- more -->

## 生命周期

### 状态

每个 Activity 至多有 4 种状态：

- 运行状态：Activity 位于返回栈栈顶（系统不回收）
- 暂停状态：Activity 不处于栈顶，但仍然可见（只有内存极低时才可能被系统回收）
- 停止状态：Activity 不处于栈顶，并不可见（系统仍然保留相应的状态和成员变量，当其他地方需要内存时才可能被回收）
- 销毁状态：Activity 从返回栈移除（系统回收）

### 生存期

为了在 Activity 生命周期的各个阶段之间导航转换，Activity 类提供六个核心回调：`onCreate()`、`onStart()`、`onResume()`、`onPause()`、`onStop()` 和 `onDestroy()`，另有一个 `onRestart()`。当 Activity 进入新状态时，系统会调用其中每个回调。

- `onCreate()`：首次创建 Activity 时调用，在其中完成初始化操作，比如加载布局、绑定事件等。会相继调用 `onStart()` 和 `onResume()` 方法
- `onStart()`：由不可见到可见时调用。会非常快速地完成，一旦此回调结束便会进入 `onResume()`
- `onResume()`：当准备好和用户交互时调用。此时 Activity 处于返回栈栈顶，并且处于运行状态
- `onPause()`：当准备去启动或恢复另一个 Activity 时调用，在其中释放消耗 CPU 的资源，保存关键数据
- `onStop()` ：当完全不可见时调用。与 `onPause()` 的区别在于：如果新活动是对话框，`onPause()` 执行，它不执行
- `onDestroy()`：在被销毁前调用，在其中释放内存。之后 Activity 变为消费状态
- `onRestart()`：由停止到运行时调用，即活动被重启

<center><img src="https://developer.android.com/guide/components/images/activity_lifecycle.png?hl=zh-cn" alt="Activity 生命周期的简化图示 " style="zoom: 80%;" /></center>

以上除了 `onRestart()` 两两对应，即：

- 完整生存期：`onCreate()`→`onDestroy()`
- 可见生存期：`onStart()`→`onStop()`
- 前台生存期：`onResume()`→`onPause()`

## 启动模式

通过 `AndroidManifest.xml` 中为 `<activity>` 指定 `android:launchMode`

### Standard

默认值。每次创建 Activity 的新实例，并将 intent 传送给该实例。

Activity 可以多次实例化，每个实例可以属于不同的任务，一个任务可以拥有多个实例。

### singleTop

在启动 Activity 时如果发现返回栈栈顶已经是该 Activity，则直接使用而不创建。

Activity 可以多次实例化，每个实例可以属于不同的任务，一个任务可以拥有多个实例（但前提是返回堆栈顶部的 Activity 不是该 Activity 的现有实例）。

### singleTask

在启动 Activity 时如果返回栈中已经有了该实例，则直接使用，并把在此 Activity 之上的所有其他 Activity 出栈，没有则新建实例。

Activity 一次只能有一个实例存在。

**注意**：虽然 Activity 在新任务中启动，但用户按**返回**按钮仍会返回到上一个 Activity。

### singleInstance

与 `"singleTask"` 相似，唯一不同的是系统不会将任何其他 Activity 启动到包含该实例的任务中。

该 Activity 始终是其任务唯一的成员；由该 Activity 启动的任何 Activity 都会在其他的任务中打开。

即有单独的返回栈管理此 Activity，不管哪个应用访问这个活动，都共用一个返回栈。

用于其他应用中调用此 Activity。

## 意图

### 显示 Intent

```kotlin
val intent = Intent(this, SecondActivity::class.java)
startActivity(intent)
```

### 隐式 Intent

在 `<activity>` 下配置 `<intent-filter>`，在 `<action>` 中指名可以相应的 `action`，`<category>` 中添加附加信息，只有 `<action>` 和 `<category>` 同时匹配，当前活动才会响应 `intent`

```xml
<activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
        <intent-filter>
            <action android:name="com.example.activitytest.ACTION_START" />
            <category android:name="android.intent.category.DEFAULT" />
        </intent-filter>
</activity>
```

```kotlin
val intent = Intent(com.example.activitytest.ACTION_START) // default 默认添加category
startActivity(intent)
```

### 向下一个 Activity 传递数据

在显示 intent 中添加：`intent.putExtra()`

### 向上一个 Activity 传递数据

`SecondActivity.kt`

```kotlin
override fun onBackPressed() {
    super.onBackPressed()
    val intent = Intent()
    intent.putExtra("data_return","msg...")
    setResult(RESULT_OK, intent)
    finish()
}
```

`FirstActivity.ky`

```kotlin
val intent = ...
startActivityForResult(intent, 1) // 参数2是请求码，唯一值即可

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    when (requestCode) {
        1 -> if (requestCode = RESULT_OK) {
            val retrunedData = data?.getStringExtra("data_return")
        }
    }
}
```
