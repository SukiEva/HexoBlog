---
title: Android四大组件之Service
date: 2021-10-01 08:18
tags:
  - Component
categories: [Framework,Android]
toc: true
updated: 2021-10-01 08:18
references:
  - title: 服务概览  | Android 开发者  | Android Developers
    url: https://developer.android.com/guide/components/services
  - title: 《第一行代码 Android 第3版》
    url: https://guolin.blog.csdn.net/article/details/105233078
---

`Service` 是一种可在后台执行长时间运行操作而不提供界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。

简单地说，服务是一种即使用户未与应用交互也可在后台运行的组件，因此，只有在需要服务时才应创建服务。

<!-- more -->

## 多线程

一般耗时的服务都需要放在子线程中运行，否则会造成主线程阻塞，从而影响用户使用。

### 基本用法

- 继承 `Thread`

	```kotlin
	class MyThread : Thread(){
	    override fun fun(){
	        // ...
	    }
	}
	// 开始运行
	MyThread().start() 
	```

- 上面的方法耦合性过高，一般使用 `Runnable` 接口

	```kotlin
	class MyThread : Runnable{
	    override fun fun(){
	        // ...
	    }
	}
	// 开始运行
	val myThread = MyThread()
	Thread(myThread).start()
	```

- 更为常见的方法是使用 `Lambda` 表达式，不需要专门定义一个类

	```kotlin
	Thread {
	    // ...
	}.start()


	// 在Kotlin中可以写为： 不需要调用 start()
	thread{
	    // ...
	}
	```

### 子线程更新 UI

Android 的 UI 是线程不安全的，直接在子线程中进行 UI 操作会出现异常。不过，Android 提供了异步消息处理机制 `Handler`。

主要包含以下部分：

- `Message`：线程间传递的消息，可以在内部携带少量消息
- `Handler`：处理者，用于发送和处理消息
- `MessageQueue`：消息队列，用于存放通过 `Handler` 发送的消息，**每个线程只有一个**
- `Looper`：每个线程中 `MessageQueue` 的管家，调用 `Looper` 的 `loop()` 后开始循环取出 `MessageQueue` 中的消息，并传递到 `Handler` 的 `handleMessage()` 中，**每个线程只有一个**

```kotlin
val updateText = 1
val handler = object : Handler(Looper.getMainLooper()) {
        override fun handleMessage(msg: Message) {
            super.handleMessage(msg)
            when (msg.what) {
                updateText -> //...
                // ...
            }
        }
    }

thread{
    val msg = Message()
    msg.what = updateText
    handler.sendMessage(msg) // 发送消息
}
```

> AsyncTask 在 API30 被 Google 废弃，转而推荐使用 [将 Kotlin 协程与生命周期感知型组件一起使用  | Android 开发者  | Android Developers](https://developer.android.com/topic/libraries/architecture/coroutines)

## Service 生命周期

最常使用的回调方法是：`onBind()`、`onCreate`、`onStartCommand`、`onDestroy`

- `onCreate`：在 Service 创建（仅第一次）的时候调用
- `onStartCommand`：在 Service 启动（每次）的时候调用
- `onDestroy`：在 Service 销毁时调用

每个 Service 只会存在一个实例

## Service 基本用法

### 启动和停止

```kotlin
val intent = Intent(this, MyService::class.java)
startService(intent) // 启动
stopService(intent) // 停止
```

### 与 Activity 通信

在活动中控制服务的具体执行，而不是启动后就无法控制。

通过 `Bind` 实现：

```kotlin
class MyService : Service() {

    private val mBinder = DownloadBinder()

    override fun onBind(intent: Intent): IBinder {
        return mBinder
    }

    class DownloadBinder : Binder() {
        fun startDownload() {
            //...
        }
    }

}
```

在活动中创建：

```kotlin
var downloadBinder = MyService.DownloadBinder()

private val connection = object:ServiceConnection{ // 匿名类
    override fun onServiceConnected(p0: ComponentName?, p1: IBinder?) { // 绑定时调用
        downloadBinder = p1 as MyService.DownloadBinder
        downloadBinder.startDownload() // 可自由选择 Service 中的方法
    }

    override fun onServiceDisconnected(p0: ComponentName?) { // 进程崩溃或被杀掉时调用，不常用
        TODO("Not yet implemented")
    }
}

val intent = Intent(this, MyService::class.java)
bindService(intent, connection, Context.BIND_AUTO_CREATE) // 绑定Service
// BIND_AUTO_CREATE: 自动创建Service，调用 onCreate，忽略 onStartCommamd
unbindService(connection) // 解绑Service
```

> 任何 Service 可以和任意 Activity 绑定，且绑定后获得相同实例

## 前台服务

前台服务是用户主动意识到的一种服务，因此在内存不足时，系统也不会考虑将其终止。

前台服务必须为状态栏提供通知，将其放在运行中的标题下方。这意味着除非将服务停止或从前台移除，否则不能清除该通知。

从 Android 9.0 开始需要申请权限 `android.permission.FOREGROUND_SERVICE`

在 `onCreate` 中：

```kotlin
val pendingIntent: PendingIntent =
        Intent(this, ExampleActivity::class.java).let { notificationIntent ->
            PendingIntent.getActivity(this, 0, notificationIntent, 0)
        }

val notification: Notification = Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
        .setContentTitle(getText(R.string.notification_title))
        .setContentText(getText(R.string.notification_message))
        .setSmallIcon(R.drawable.icon)
        .setContentIntent(pendingIntent)
        .setTicker(getText(R.string.ticker_text))
        .build()

startForeground(ONGOING_NOTIFICATION_ID, notification)
```

## IntentService

用来执行耗时操作的 `Service` 的子类。

实现 `onHandleIntent()`，该方法会接收每个启动请求的 Intent，以便您执行后台工作。

```kotlin
/**
 * A constructor is required, and must call the super [android.app.IntentService.IntentService]
 * constructor with a name for the worker thread.
 */
class HelloIntentService : IntentService("HelloIntentService") {

    /**
     * The IntentService calls this method from the default worker thread with
     * the intent that started the service. When this method returns, IntentService
     * stops the service, as appropriate.
     */
    override fun onHandleIntent(intent: Intent?) {
        // Normally we would do some work here, like download a file.
        // For our sample, we just sleep for 5 seconds.
        try {
            Thread.sleep(5000)
        } catch (e: InterruptedException) {
            // Restore interrupt status.
            Thread.currentThread().interrupt()
        }

    }
}
```

重写其他回调方法，（如 `onCreate()`、`onStartCommand()` 或 `onDestroy()`），请确保调用超类实现

例如，`onStartCommand()` 必须返回默认实现，即如何将 Intent 传递给 `onHandleIntent()`：

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show()
    return super.onStartCommand(intent, flags, startId)
}
```

启动

```kotlin
Intent(this, HelloService::class.java).also { intent ->
    startService(intent)
}
```

### 扩展服务类

借助 `IntentService`，您可以非常轻松地实现启动服务。但是，若要求服务执行多线程（而非通过工作队列处理启动请求），则可通过扩展 `Service` 类来处理每个 Intent。

为进行比较，以下示例代码展示了 `Service` 类的实现，该类执行的工作与上述使用 `IntentService` 的示例完全相同。换言之，对于每个启动请求，其均使用工作线程来执行作业，且每次仅处理一个请求。

```kotlin
class HelloService : Service() {

    private var serviceLooper: Looper? = null
    private var serviceHandler: ServiceHandler? = null

    // Handler that receives messages from the thread
    private inner class ServiceHandler(looper: Looper) : Handler(looper) {

        override fun handleMessage(msg: Message) {
            // Normally we would do some work here, like download a file.
            // For our sample, we just sleep for 5 seconds.
            try {
                Thread.sleep(5000)
            } catch (e: InterruptedException) {
                // Restore interrupt status.
                Thread.currentThread().interrupt()
            }

            // Stop the service using the startId, so that we don't stop
            // the service in the middle of handling another job
            stopSelf(msg.arg1)
        }
    }

    override fun onCreate() {
        // Start up the thread running the service.  Note that we create a
        // separate thread because the service normally runs in the process's
        // main thread, which we don't want to block.  We also make it
        // background priority so CPU-intensive work will not disrupt our UI.
        HandlerThread("ServiceStartArguments", Process.THREAD_PRIORITY_BACKGROUND).apply {
            start()

            // Get the HandlerThread's Looper and use it for our Handler
            serviceLooper = looper
            serviceHandler = ServiceHandler(looper)
        }
    }

    override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
        Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show()

        // For each start request, send a message to start a job and deliver the
        // start ID so we know which request we're stopping when we finish the job
        serviceHandler?.obtainMessage()?.also { msg ->
            msg.arg1 = startId
            serviceHandler?.sendMessage(msg)
        }

        // If we get killed, after returning from here, restart
        return START_STICKY
    }

    override fun onBind(intent: Intent): IBinder? {
        // We don't provide binding, so return null
        return null
    }

    override fun onDestroy() {
        Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show()
    }
}
```
