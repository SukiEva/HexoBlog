---
title: Kotlin
date: 2021-10-01 19:45
tags:
  - Kotlin
categories: [Programming Language,Kotlin]
toc: true
updated: 2021-10-01 19:45
references:
  - title: Android 上的 Kotlin 协程  | Android 开发者  | Android Developers
    url: https://developer.android.com/kotlin/coroutines?hl=zh-cn
  - title: Android官方架构组件:Lifecycle详解&原理分析_却把清梅嗅的博客-CSDN博客_lifecycle原理
    url: https://qingmei2.blog.csdn.net/article/details/79029657
---

Kotlin 是一种在 Java 虚拟机上运行的静态类型编程语言，被称之为 Android 世界的 Swift，由 JetBrains 设计开发并开源。

Kotlin 可以编译成 Java 字节码，也可以编译成 JavaScript，方便在没有 JVM 的设备上运行。

~~用了 Kotlin 就不想回到 Java，本文主要记录一些高级用法~~

<!-- more -->

更多细节可查看：[基本语法 - Kotlin 语言中文站 (kotlincn.net)](https://www.kotlincn.net/docs/reference/basic-syntax.html)

## 基本知识

### 区间

- 左闭右闭：`0..10`
- 左闭右开：`0 unitil 10 (step 2)`
- 降序（左闭右闭）：`10 downto 1`

### 类

#### 主构造函数

```kotlin
class Examle(...){
    init{
        ...
    }
}
```

#### 次构造函数

不常用，一般直接指定默认值

```kotlin
class Examle(...){
    constructor(...) : this(...){}
    sonstructor() : this (...){}
}
```

#### 修饰符

- `public`：（默认）对所有类可见
- `private`：对当前类内部可见
- `protected`：对当前类和子类可见
- `internal`：对同一模块的类可见

#### 数据类和单例类

- 数据类：`data class`，等同于 Java 中一长串的 bean
- 单例类：`object`，等同于 Java 中的单例模式（全局至多只有一个实例）

### Lambda

简而言之，就是可以作为参数传递的代码。

太常用也太多了，写写就会了

## 标准函数

只介绍最常用的 3 个

### With

一个非扩展函数：**上下文对象**作为参数传递，但是在 lambda 表达式内部，它可以作为接收者（`this`）使用。 **返回值**是 lambda 表达式结果。

接收 2 个参数：

- 任意类型的对象
- Lambda 表达式

```kotlin
val result = with(obj){
    // obj 上下文
    "value" // with 返回值
}

val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}
```

### Run

**上下文对象** 作为接收者（`this`）来访问。 **返回值** 是 lambda 表达式结果。

`run` 和 `with` 做同样的事情，但是调用方式和 `let` 一样——作为上下文对象的扩展函数.

当 lambda 表达式同时包含对象初始化和返回值的计算时，`run` 很有用。

```kotlin
val result = service.run {
    port = 8080
    query(prepareRequest() + " to port $port")
}

// 同样的代码如果用 let() 函数来写:
val letResult = service.let {
    it.port = 8080
    it.query(it.prepareRequest() + " to port ${it.port}")
}
```

### Apply

**上下文对象** 作为接收者（`this`）来访问。 **返回值** 是上下文对象本身。

对于不返回值且主要在接收者（`this`）对象的成员上运行的代码块使用 `apply`。`apply` 的常见情况是对象配置。这样的调用可以理解为“将以下赋值操作应用于对象”。

```kotlin
val adam = Person("Adam").apply {
    age = 32
    city = "London"        
}
println(adam)
```

### 函数选择

为了选择合适的作用域函数，它们之间的主要区别表。

| 函数      | 对象引用   | 返回值          | 是否是扩展函数       |
| :------ | :----- | :----------- | :------------ |
| `let`   | `it`   | Lambda 表达式结果 | 是             |
| `run`   | `this` | Lambda 表达式结果 | 是             |
| `run`   | -      | Lambda 表达式结果 | 不是：调用无需上下文对象  |
| `with`  | `this` | Lambda 表达式结果 | 不是：把上下文对象当做参数 |
| `apply` | `this` | 上下文对象        | 是             |
| `also`  | `it`   | 上下文对象        | 是             |

## 拓展函数和运算符重载

### 拓展函数

基本格式：

```kotlin
fun ClassName.methodName(pararm: Int):Int{
    return 0
}
```

例子：

```kotlin
fun String.showToast(duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeToast(MyApp.context, this, duration).show()
}
```

### 运算符重载

例子：

```kotlin
class Money(val value:Int){
    operator fun plus(money:Money):Money{
        return Money(value+money.value)
    }
}
```

## 高阶函数和内联函数

### 高阶函数

如果一个函数接收另一个函数作为参数，或者韩非子的类型是另一个函数，则称之为高阶函数。

基本规则：`() -> Unit`

例子如下，相当于能嵌套函数

```kotlin
fun CardItem(
    onClick: () -> Unit = {}
) 
```

### 内联函数

高阶函数实现的 Lambda 表达式在底层被转换为匿名类，每次调用都会创建一个新的匿名类实例，造成额外开销

函数前添加 `inline` 可消除开销：`inline fun example(){}`

## 泛型和委托

### 泛型

一般编程模式下，需要给任何一个变量指定具体的类型，而泛型允许我们在不指定具体类型的情况下进行编程，这样的代码会有更好的拓展性。

**泛型类：**

```kotlin
class MyClass<T>{
    fun method(param: T): T{
        return param
    }
}

val myClass = MyClass<Int>() // 指定 Int 泛型
val result = myClass.method(123) // 得到 Int 返回值
```

**泛型方法：**

```kotlin
class MyClass{
    fun <T> method(param: T): T{
        return param
    }
}

val myClass = MyClass()
val result = myClass.method<Int>(123)

// kotlin 可以自动识别
val result = myClass.method(123)
```

**范围：**

可通过 `<T : Number>` 的形式指定范围，此时只允许数字类型，字符串会报错

默认类型可空，即 `Any?`，不想为空可改为 `Any`

### 委托

委托是一种设计模式，操作对象自己不会去处理某段逻辑，而是把工作委托给另外一个辅助对象处理。

**类委托：**

借助委托可以轻松自己实现类，以下通过 `Hashset` 自定义一个类：`by` 是实现委托的关键字

```kotlin
class MySet<T>(val helperSet: HashSet<T>) : Set<T> by helperSet{
    fun helloworld() = println("hello world")
    override fun isEmpty = false
}
```

**属性委托：**

将一个属性（字段）的具体实现委托给另一个类去完成

```kotlin
class MyClass{
    val p by Delegate()
}

// 调用 p 时，自动调用 Delegate类的 getValue 方法，赋值时调用 setValue 方法
// 以下是标准的实现
class Delegate{
    var propValue: Any? = null
    
    // 参数1: 指定该类的委托在什么类可以使用
    // 参数2: Kotlin属性操作类，可用于获取各种属性相关的值
    operator fun getValue(myClass: MyClass, prop: Kproperty<*>): Any?{
        return propValue
    }
    
    operator fun setValue(myClass: MyClass, prop: Kproperty<*>, value: Any?): Any?{
        propValue = value
    }
    
}
```

### 泛型实化

指定泛型的实际类型

`inline fun <reified T> start(block: Intent.() -> Unit = {}){}`

此时就可以得到 `T::class.java` 类型：

```kotlin
val intent = Intent(context, T::class.java)
```

### 泛型协变和逆变

> Todo

## 协程

可以理解为轻量级的线程，可以在单线程中模拟多线程效果，与线程不同点在于：

- 线程依靠操作系统调度实现不同线程切换
- 协程在编程语言层面实现不同协程切换，大大提高并发编程的运行效率

特点：协程是我们在 Android 上进行异步编程的推荐解决方案。值得关注的特点包括：

- **轻量**：可以在单个线程上运行多个协程，因为协程支持挂起，不会使正在运行协程的线程阻塞。挂起比阻塞节省内存，且支持多个并行操作。
- **内存泄漏更少**：使用结构化并发机制在一个作用域内执行多项操作。
- **内置取消支持**：取消操作会自动在运行中的整个协程层次结构内传播。
- **Jetpack 集成**：许多 Jetpack 库都包含提供全面协程支持的扩展。某些库还提供自己的协程作用域，可供您用于结构化并发。

### 作用域构建器

**Tips：**

- `delay()`：让当前协程延迟指定时间再运行，会阻塞挂起函数，只会挂起当前协程，不会影响其他协程执行
- `suspend`：将任意函数声明为挂起函数，挂起函数间可以互相调用

**作用域**

- `GlobalScope.launch{}`：顶层协程（不建议用）
- `runBlocking{}`：作用域内所有代码和子协程没有全部执行完之前一直阻塞当前线程（测试用，生产可能有性能问题）
- `launch`：在作用域内创建多个协程
- `coroutineScope{}`：继承外部协程作用域并创建一个子协程，配合 `suspend` 使用，和 `runBlocking` 类似，用于生产环境

```kotlin
suspend fun printDot() = coroutineScope{
	launch{}
}
```

**常用写法**

```kotlin
val job = Job()
val scope = CoroutineScope(job)
scope.launch{
    //...
}
job.cancel() // 取消作用域内所有协程
```

### 获取执行结果

调用 `async` 后，代码立即执行，如果调用 `await()` 时还没执行完，则会阻塞当前协程，直到获得 `async` 执行结果

```kotlin
runBlocking{
    val result = async{
        5+5
    }.await()
}
```

`withContext` 是一个挂起函数，可以理解为 `async` 的一种简化版写法：

```kotlin
runBlocking{
    val result = withContext(Dispatchers.Default){
        5+5
    }
}
```

### 线程参数

除了 `coroutineScope`，其他函数都可以指定线程参数，`withContext` 必须，其他可选

- `Dispatchers.Default`：默认低并发线程策略
- `Dispatchers.IO`：较高并发的线程策略
- `Dispatchers.Main`：不会开启子线程，在 Android 主线程执行（只在安卓中使用）

> Android 中要求网络请求必须在线程执行，定义协程也不行
