---
title: Android四大组件之BroadcastReceiver
date: 2021-09-28 19:37
tags:
  - Component
categories: [Framework,Android]
toc: true
updated: 2021-09-28 19:37
references:
  - title: 广播概览  | Android 开发者  | Android Developers
    url: https://developer.android.com/guide/components/broadcasts?hl=zh-cn
  - title: 《第一行代码 Android 第3版》
    url: https://guolin.blog.csdn.net/article/details/105233078
---

Android 应用与 Android 系统和其他 Android 应用之间可以相互收发广播消息，这与 [发布-订阅](https://en.wikipedia.org/wiki/Publish–subscribe_pattern) 设计模式相似。这些广播会在所关注的事件发生时发送。

一般来说，广播可作为跨应用和普通用户流之外的消息传递系统。

<!-- more -->

## 广播类型

- 标准广播：异步执行，所有的 BroadcastReceiver 均会收到消息，没有先后顺序。
	- 优点：效率高
	- 缺点：无法被截断
- 系统广播：同步执行，同一时刻只有一个 BroadcastReceiver 能收到消息，逻辑执行完毕才会继续传递，有先后顺序。
	- 优点：优先级高的 BroadcastReceiver 能先收到消息，并能截断广播
	- 缺点：效率没有标准广播高

## 系统广播

Android 内置很多系统级别的广播，可以监听这些广播来得到各种系统状态信息。

可以在 `<Android SDK>/platforms/<任意api>/data/broadcast_actions.txt` 中查看

### 动态注册

程序启动之后才能接收广播

```kotlin
lateinit var timeChangeReceiver: TimeChangeReceiver

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    val intentFilter = IntentFilter()
    intentFilter.addAction("android.intent.action.TIME_TICK") // 系统广播动作
    timeChangeReceiver = TimeChangeReceiver()
    registerReceiver(timeChangeReceiver, intentFilter) // 动态注册
}


override fun onDestroy() {
    super.onDestroy()
    unregisterReceiver(timeChangeReceiver) // 取消注册
}

// 内部类注册
inner class TimeChangeReceiver : BroadcastReceiver() { // 继承 BroadcastReceiver
    override fun onReceive(p0: Context?, p1: Intent?) {
        // 不允许多线程，所以不能执行耗时操作
        p0?.let {
            "Time has changed".showToast(it)
        }
    }
}
```

### 静态注册

由于大量恶意程序利用静态注册机制在程序未启动时监听系统广播，从而使任意应用都可以频繁从后台唤醒，因此 Android 每个版本都在削减静态注册 BroadcastReceiver 的功能。

Android 8.0 以后，所有隐式广播（没有指定发送给哪个程序的广播，大多数系统广播都属于此）都不允许使用静态注册接收。

例外情况见 [隐式广播例外情况  | Android 开发者  | Android Developers](https://developer.android.com/guide/components/broadcast-exceptions?hl=zh-cn)

**注意**：尽管这些隐式广播仍在后台运行，但应避免为其注册监听器

```xml
    <receiver android:name=".MyBroadcastReceiver"  android:exported="true">
        <intent-filter>
            <!-- 需要权限 android.permission.RECEIVE_BOOT_COMPLETED-->
            <action android:name="android.intent.action.BOOT_COMPLETED"/> 
            <action android:name="android.intent.action.INPUT_METHOD_CHANGED" />
        </intent-filter>
    </receiver>
```

## 自定义广播

<br>

### 发送标准广播

#### 创建 BroadcastReceiver

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(p0: Context?, p1: Intent?) {
        // TODO: 2021/9/28  
    }
}
```

#### 注册广播

```xml
    <receiver android:name=".MyBroadcastReceiver"  android:exported="true">
        <intent-filter>
            <action android:name="com.example.MY_BROADCAST"/> 
        </intent-filter>
    </receiver>
```

#### 发送广播

```kotlin
val intent = Intent("com.example.MY_BROADCAST")
intent.setPackage(packageName) // 指定包名，使其成为显示广播
sendBroadcast(intent)
```

### 发送有序广播

前面与标准广播相同，发送广播时选择 `sendOrderedBroadcast(intent,null)` （参数 2 是与权限相关的字符串）

#### 设置优先级

因为有序广播可以被截断，在静态注册时指定优先级：

`<intent-filter android:priority="100">`

#### 截断

使用 `abortBroadcast()`

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(p0: Context?, p1: Intent?) {
        // TODO: 2021/9/28  
        abortBroadcast()
    }
}
```
