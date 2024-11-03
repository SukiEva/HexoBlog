---
title: Golang
tags:
  - Go
categories: [Programming Language,Go]
toc: true
date: 2022-01-19 15:30
updated: 2022-01-19 15:30
references:
  - title: Tony Bai · Go 语言第一课
    url: https://time.geekbang.org/column/intro/100093501?tab=catalog
  - title: GitH《The Way to Go》中文译本，中文正式名《Go 入门指南》
    url: https://github.com/unknwon/the-way-to-go_ZH_CN
  - title: 《Go Web 编程》| Go 技术论坛
    url: https://learnku.com/docs/build-web-application-with-golang/about-this-book/3151
---

Go（又称 Golang）是 Google 开发的一种静态强类型、编译型、并发型，并具有垃圾回收功能的编程语言。

<!-- more -->

## 基础

### 关键字

Go 是一门类似 C 的编译型语言，但是它的编译速度非常快。

这门语言的关键字总共也就二十五个：

```go
break    default      func    interface    select
case     defer        go      map          struct
chan     else         goto    package      switch
const    fallthrough  if      range        type
continue for          import  return       var
```

### 变量

一个特定的名字与位于特定位置的内存块绑定在一起，这个名字被称为**变量**。

声明变量的一般形式是使用 `var` 关键字：`var identifier type`

> 变量的类型放在变量的名称之后，是为了避免像 C 语言中那样含糊不清的声明形式。
>
> C++：int* a, b; 	// a 是指针，b 不是
>
> Go：var a,b *int	// a，b 都是指针

```go
var a int
var b bool
var str string

// 因式分解关键字的写法，一般用于声明全局变量
var (
	a int
	b bool
	str string
)

// 声明与赋值（初始化）语句也可以组合起来
var identifier [type] = value
var a int = 15
var i = 5 // 自动推断
var b bool = false
var str string = "Go says hello to the world!"

// 短变量声明，用于函数体内声明局部变量
a := 1
a, b, c := 5, 7, "abc"
a, b = b, a  // 相当于C++中的 swap(a,b)
```

> 变量的命名规则遵循骆驼命名法，即首个单词小写，每个新单词的首字母大写
>
> 如果全局变量希望能够被外部包所使用，则需要将首个单词的首字母也大写

#### 值类型

- 所有像 int、float、bool 和 string 这些基本类型都属于值类型，使用这些类型的变量直接指向存在内存中的值
- 像数组和结构体这些复合类型也是值类型
- 当使用等号 `=` 将一个变量的值赋值给另一个变量时，如：`j = i`，实际上是在内存中将 i 的值进行了拷贝
- 可以通过 `&i` 来获取变量 i 的内存地址（每次的地址都可能不一样），值类型的变量的值存储在栈中

#### 引用类型

- 一个引用类型的变量 r1 存储的是 r1 的值所在的内存地址（数字），或内存地址中第一个字所在的位置（这个内存地址被称之为指针）
- 在 Go 语言中，指针、slices（切片）、maps 和 channel 都属于引用类型
- 当使用赋值语句 `r2 = r1` 时，只有引用（地址）被复制
- 当 r1 的值被改变了，那么这个值的所有引用都会指向被修改后的内容
- 被引用的变量会存储在堆中，以便进行垃圾回收，且比栈拥有更大的内存空间

### 常量

Go 语言的常量是一种在源码编译期间被创建的语法元素。

```go
const Pi float64 = 3.14159265358979323846 // 单行常量声明，显示
const Pi = 3.14159 // 隐式

// 以const代码块形式声明常量
const (
    size int64 = 4096
    i, j, s = 13, 14, "bar" // 单行声明多个常量
)
```

> 常量的类型只局限于基本数据类型，包括数值类型、字符串类型、布尔类型

Go 语言在常量方面的创新包括下面这几点：

- 支持无类型常量：

	可以不显示指定类型，比如 `const n = 13`

- 支持隐式自动转型：

	对于无类型常量参与的表达式求值，Go 编译器会根据上下文中的类型信息，把无类型常量自动转换为相应的类型后，再参与求值计算，这一转型动作是隐式进行的

- 可用于实现枚举：
	- 隐式重复前一个非空表达式

		```go
		const (
		    Apple, Banana = 11, 22
		    Strawberry, Grape  = 11, 22 // 使用上一行的初始化表达式
		    Pear, Watermelon  = 11, 22 // 使用上一行的初始化表达式
		)
		```

	- iota 是一个预定义标识符，可以从 0 开始自增（位于同一行的 iota 即便出现多次，多个 iota 的值也是一样的）

		```go
		const (
		    Apple, Banana = iota, iota + 10 // 0, 10 (iota = 0)
		    Strawberry, Grape // 1, 11 (iota = 1)
		    Pear, Watermelon  // 2, 12 (iota = 2)
		)
		// 如果想从 1 开始
		// _ = iota // 0 
		// 每遇到一次 const 关键字，iota 就重置为 0
		```

### 数组

数组是一个长度固定的、由同构类型元素组成的连续序列，包含两个重要属性：元素的类型和数组长度（元素的个数）

数组变量声明：

```go
var arr [5]int // 一维
var mArr [2][3][4]int // 多维
var arr2 = [6]int { 11, 12, 13, 14, 15, 16,} // [11 12 13 14 15 16]
var arr3 = [...]int { 21, 22, 23,} // [21 22 23] ... 自动计算元素个数
```

> 数组类型变量是一个整体，这就意味着一个数组变量表示的是整个数组。
>
> 这点与 C 语言完全不同，在 C 语言中，数组变量可视为指向数组第一个元素的指针。

### 切片

在 Go 语言中，数组更多是“退居幕后”，承担的是底层存储空间的角色。

切片就是数组的“描述符”，也正是因为这一特性，切片才能在函数参数传递时避免较大性能开销。可以说，**切片之于数组就像是文件描述符之于文件**。

去掉“长度”这一束缚后，切片展现出更为灵活的特性

```go
var nums = []int{1, 2, 3, 4, 5, 6}
```

#### 底层实现

切片的**底层数据结构**：在运行时其实是一个三元组结构

```go
type slice struct {
    array unsafe.Pointer // 是指向底层数组的指针
    len   int // 切片的长度，即切片中当前元素的个数
    cap   int // 底层数组的长度，也是切片的最大容量，cap 值永远大于等于 len 值
} 
```

> Go 编译器会自动为每个新创建的切片，建立一个底层数组，默认底层数组的长度与切片初始元素个数相同

#### 创建

切片的创建根据情况不同，主要通过以下 3 种方法创建：

```go
// 1、make 函数
sl := make([]byte, 6, 10) // 其中10为cap值，即底层数组长度，6为切片的初始长度
sl := make([]byte, 6) // cap = len = 6
// 2、数组切片化 array[low : high : max]
arr := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
sl := arr[3:7:9] // len = high - low ; cap = max - low ; 即数组下标 [3,7)
// 3、切片创建切片 与 方法2 书写方法相同
```

#### 动态扩容

当通过 append 操作向切片追加数据的时候，如果这时切片的 len 值和 cap 值是相等的，也就是说切片底层数组已经没有空闲空间再来存储追加的值了，Go 运行时就会对这个切片做扩容操作，来保证切片始终能存储下追加的新值。

```go
var s []int
s = append(s, 11) 
fmt.Println(len(s), cap(s)) //1 1	创建底层数组 u1（长度1）
s = append(s, 12) 
fmt.Println(len(s), cap(s)) //2 2	创建底层数组 u2（长度2 = u1的两倍），拷贝 u1 元素， array 指向 u2   
s = append(s, 13) 
fmt.Println(len(s), cap(s)) //3 4	创建底层数组 u3（长度4 = u2的两倍），拷贝 u2 元素， array 指向 u3
s = append(s, 14) 
fmt.Println(len(s), cap(s)) //4 4	cap 足够
s = append(s, 15) 
fmt.Println(len(s), cap(s)) //5 8   创建底层数组 u4（长度8 = u3的两倍），拷贝 u3 元素， array 指向 u4
```

### Map

map 是 Go 语言提供的一种抽象数据类型，它表示一组无序的键值对。

形式：`map[key_type]value_type`

> Go 语言中要求，key 的类型必须支持“==”和“!=”两种比较操作符。
>
> 函数类型、map 类型自身，以及切片类型是不能作为 map 的 key 类型的。

#### 底层实现

Go 运行时使用一张**哈希表**来实现抽象的 map 类型。

运行时实现了 map 类型操作的所有功能，包括查找、插入、删除等。

在编译阶段，Go 编译器会将 Go 语法层面的 map 操作，重写成运行时对应的函数调用。

```go
// 创建map类型变量实例
m := make(map[keyType]valType, capacityhint) → m := runtime.makemap(maptype, capacityhint, m)

// 插入新键值对或给键重新赋值
m["key"] = "value" → v := runtime.mapassign(maptype, m, "key") v是用于后续存储value的空间的地址

// 获取某键的值 
v := m["key"]      → v := runtime.mapaccess1(maptype, m, "key")
v, ok := m["key"]  → v, ok := runtime.mapaccess2(maptype, m, "key")

// 删除某键
delete(m, "key")   → runtime.mapdelete(maptype, m, “key”)
```

`hmap` 类型是 `map` 类型的头部结构（header），之前提到的 map 类型的描述符，它存储了后续 map 类型操作所需的所有信息

> 不要依赖 map 的元素遍历顺序；
>
> map 不是线程安全的，不支持并发读写；
>
> 不要尝试获取 map 中元素（value）的地址

**map 扩容：**

当 count > LoadFactor * 2^B 或 overflow bucket 过多时，运行时会自动对 map 进行扩容（ Go 最新 1.17 版本 LoadFactor 设置为 6.5）

#### 初始化

切片类型，初值为零值 nil 的切片类型变量，可以借助内置的 append 的函数进行操作，这种在 Go 语言中被称为“零值可用”

map 类型，因为它内部实现的复杂性，无法“零值可用”

```go
var m map[string]int // m = nil
m["key"] = 1         // 发生运行时异常：panic: assignment to entry in nil map

// 1、使用复合字面值初始化 map 类型变量，
m := map[int]string{}

type Position struct {     
    x float64     
    y float64
}
// Go 允许省略字面值中的元素类型
m2 := map[Position]string{
    Position{29.935523, 52.568915}: "school", // Postion 可省略，如下
    {25.352594, 113.304361}: "shopping-mall",
    {73.224455, 111.804306}: "hospital",
}

// 2、make 函数
m1 := make(map[int]string) // 未指定初始容量
m2 := make(map[int]string, 8) // 指定初始容量为8
```

#### 基本操作

和切片类型一样，map 也是引用类型。

这就意味着 map 类型变量作为参数被传递给函数或方法的时候，实质上传递的只是一个“描述符”，而不是整个 map 的数据拷贝，所以这个传递的开销是固定的，而且也很小。

```go
// 插入，更新相同
m := make(map[int]string)
m[1] = "value1"
// 查找
v := m["key1"]
v, ok := m["key1"]
if !ok { 
    // "key1"不在map中
}
// 删除
delete(m, "key2") // 删除"key2"
// 遍历，对同一 map 做多次遍历的时候，每次遍历元素的次序都不相同
for k, v := range m { 
    fmt.Printf("[%d, %d] ", k, v) 
}
```

## 控制结构

### If-else

```go
if condition1 {
	// do something	
} else if condition2 {
	// do something else	
} else {
	// catch-all or default
}

// 多返回值简化
if value, ok := readData(); ok {
    // do something
}
```

### Switch

```go
switch var1 {
	case val1:
		...
	case val2:
		...
	default:
		...
}

func checkWorkday(a int) {
    switch a {
    case 1, 2, 3, 4, 5:
        println("it is a work day")
    case 6, 7:
        println("it is a weekend day")
    default:
        println("are you live on earth")
    }
}

//如果需要执行下一个 case 的代码逻辑，可以显式使用 Go 提供的关键字 fallthrough 来实现
switch switchexpr() { 
    case case1(): 
    	println("exec case1") 
    	fallthrough 
    case case2(): 
    	println("exec case2") 
    	fallthrough 
    default: 
    	println("exec default") }
```

### For

Go 语言不提供 while，所以 for 循环更加强大

```go
for i := 0; i < 5; i++ {
	fmt.Printf("This is the %d iteration\n", i)
}

for i >= 0 {
	i = i - 1
	fmt.Printf("The variable i is now: %d\n", i)
}
```

### For-range

遍历数组、切片、字符串、map 等的好帮手

```go
for i, v := range sl {
    fmt.Printf("sl[%d] = %d\n", i, v)
}
```

> break、continue 与其他语言功能相同
>
> label +goto 会造成可读性极差，不推荐使用（基本没看到人用）

## 函数

Go 函数支持多返回值，函数定义一般如下形式：

```go
func funcName(参数列表) 返回值列表

func foo() // 无返回值
func foo() error // 仅有一个返回值
func foo() (int, string, error) // 有2或2个以上返回值
```

函数在 Go 语言中属于“一等公民（First-Class Citizen）”：

- Go 函数可以存储在变量中，且拥有自己的类型

```go
var dfs func(index int) // 函数类型
dfs = func(index int) {	// 匿名函数，闭包
    // ...
}
```

- 支持在函数内创建并通过返回值返回

```go
func dfs(task string) func() { 
    return func() {  
        // ...
    }
}
```

- 作为参数传入函数

```go
time.AfterFunc(time.Second*2, func() { println("timer fired") })
```

### 参数传递

#### 按值传递

> call by value

Go 默认使用**按值传递**来传递参数，也就是传递参数的**副本**（逐位拷贝）。

- 函数接收参数副本之后，在使用变量的过程中可能对副本的值进行更改，但不会影响到原来的变量

#### 引用传递

> call by reference

如果希望函数可以直接修改参数的值，而不是对参数的副本进行操作，

需要将参数的地址（变量名前面添加&符号，比如 &variable）传递给函数，这就是**引用传递**。

- 传递指针（一个 32 位或者 64 位的值）的消耗都比传递副本来得少

#### 变长参数

如果函数的最后一个参数是采用 `…type` 的形式，那么这个函数就可以处理一个变长的参数，这个长度可以为 0，这样的函数称为变参函数。

```go
func Greeting(prefix string, who ...string)
Greeting("hello:", "Joe", "Anna", "Eileen")
```

### 命名返回值

**尽量使用命名返回值：会使代码更清晰、更简短，同时更加容易读懂。**

```go
func getX2AndX3(input int) (int, int) { // 非命名
    return 2 * input, 3 * input
}

func getX2AndX3_2(input int) (x2 int, x3 int) { // 命名
    x2 = 2 * input
    x3 = 3 * input
    // return x2, x3
    return
}
```

### Defer

defer 是 Go 语言提供的一种延迟调用机制，defer 的运作离不开函数。

- 在 Go 中，只有在函数（和方法）内部才能使用 defer
- defer 关键字后面只能接函数（或方法），这些函数被称为 deferred 函数。

	defer 将它们注册到其所在 Goroutine 中，用于存放 deferred 函数的栈数据结构中，这些 deferred 函数将在执行 defer 的函数退出前，按后进先出（LIFO）的顺序被程序调度执行。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202201191633765.png" style="zoom: 50%;" />

关键字 defer 允许我们推迟到函数返回之前（或任意位置执行 `return` 语句之后）一刻才执行某个语句或函数。

所以，deferred 函数是一个可以在任何情况下为函数进行收尾工作的好“伙伴”。

```go
// 关闭文件流
// open a file  
defer file.Close()
// 解锁加锁的资源
mu.Lock()  
defer mu.Unlock()
// 打印最终报告
printHeader()  
defer printFooter()
// 关闭数据库连接
// open a database connection  
defer disconnectFromDB()
```

#### 跟踪

使用 defer 可以跟踪函数的执行过程

```go
func Trace(name string) func() { 
	println("enter:", name) 
	return func() { 
		println("exit:", name) 
	}
}

func foo() { 
    defer Trace("foo")() 
    bar()
}

func bar() { 
    defer Trace("bar")()
}

func main() { 
    defer Trace("main")() 
    foo()
}
```

输出：

```
enter: main
enter: foo
enter: bar
exit: bar
exit: foo
exit: main
```

### 内置函数

<table>
<thead>
<tr>
<th>名称</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>close</td>
<td>用于管道通信</td>
</tr>
<tr>
<td>len、cap</td>
<td>len 用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；cap 是容量的意思，用于返回某个类型的最大容量（只能用于数组、切片和管道，不能用于 map）</td>
</tr>
<tr>
<td>new、make</td>
<td>new 和 make 均是用于分配内存：new 用于值类型和用户定义的类型，如自定义结构，make 用于内置引用类型（切片、map 和管道）。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针（详见第 10.1 节）。它也可以被用于基本类型：<code>v := new(int)</code>。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作（详见第 7.2.3/4 节、第 8.1.1 节和第 14.2.1 节）<strong>new() 是一个函数，不要忘记它的括号</strong></td>
</tr>
<tr>
<td>copy、append</td>
<td>用于复制和连接切片</td>
</tr>
<tr>
<td>panic、recover</td>
<td>两者均用于错误处理机制</td>
</tr>
<tr>
<td>print、println</td>
<td>底层打印函数（详见第 4.2 节），在部署环境中建议使用 fmt 包</td>
</tr>
<tr>
<td>complex、real imag</td>
<td>用于创建和操作复数（详见第 4.5.2.2 节）</td>
</tr>
</tbody>
</table>

## 结构与方法

### Struct

结构体定义的一般方式如下：

```go
type identifier struct {
    field1 type1
    field2 type2
    ...
}

type Person struct { 
    Name string 
    Phone string
    Addr string
}
// 可以无需提供字段的名字，只需要使用其类型就可以了
type Book struct { 
    Title string 
    Person  // 嵌入字段（匿名字段）
}
```

`type T struct {a, b int}` 也是合法的语法，它更适用于简单的结构体。

#### 初始化

```go
// 1-struct as a value type:
var pers1 Person
pers1.firstName = "Chris"
pers1.lastName = "Woodward"
upPerson(&pers1)

// 2—struct as a pointer:
pers2 := new(Person)
pers2.firstName = "Chris"
pers2.lastName = "Woodward"
(*pers2).lastName = "Woodward"  // 这是合法的
upPerson(pers2)

// 3—struct as a literal:
pers3 := &Person{"Chris","Woodward"}
upPerson(pers3)

// 更常用：field:value 形式的复合字面值
t := &Timer{ 
    C: c, 
    r: runtimeTimer{ 
        when: when(d), 
    	f: sendTime, 
        arg: c, 
	}, 
}
```

### Method

在 Go 语言中，结构体就像是类的一种简化形式，那么类的方法在哪里呢？

Go 方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量。因此方法是一种特殊类型的函数。

定义方法的一般格式如下：

```go
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

- 方法声明要与 receiver 参数的基类型声明放在同一个包内。（即 struct 与 method 在同一个 package）
- receiver 参数的基类型本身不能为指针类型或接口类型

```go
type TwoInts struct {
	a int
	b int
}

func (tn *TwoInts) AddThem() int {
	return tn.a + tn.b
}

func (tn *TwoInts) AddToParam(param int) int {
	return tn.a + tn.b + param
}
```

#### Receiver 参数的选择

- 如果 Go 方法要把对 receiver 参数代表的类型实例的修改，反映到原类型实例上，那么我们应该选择 *T 作为 receiver 参数的类型
- 如果 receiver 参数类型的 size 较大，以值拷贝形式传入就会导致较大的性能开销，这时我们选择 *T 作为 receiver 类型可能更好些
- T 类型是否需要实现某个接口，也就是是否存在将 T 类型的变量赋值给某接口类型变量的情况。
  - 如果需要，那使用 T 作为 receiver 参数的类型，来满足接口类型方法集合中的所有方法
  - 如果 T 不需要，但 `*T` 需要，`*T` 的方法集合是包含 T 的方法集合的，参考上面 2 个原则选择

**总结：**

> - 类型 T 的可调用方法集包含接受者为 *T 或 T 的所有方法集
> - 类型 *T 的可调用方法集包含接受者为 *T 的所有方法
> - 类型 *T 的可调用方法集不包含接受者为 T 的方法

## 接口与反射

### Interface

Go 语言不是一种 *“传统”* 的面向对象编程语言：它里面没有类和继承的概念。

但是 Go 语言里有非常灵活的 **接口** 概念，通过它可以实现很多面向对象的特性。

接口提供了一种方式来 **说明** 对象的行为：如果谁能搞定这件事，它就可以用在这儿。

接口定义了一组方法（**方法集**），但是这些方法不包含（实现）代码：它们没有被实现（它们是抽象的）。接口里也不能包含变量。

通过如下格式定义接口：

```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

上面的 `Namer` 是一个 **接口类型**。

**命名：**

- 接口的名字由方法名加 `er` 后缀组成
- 当后缀 `er` 不合适时，以 `able` 结尾或者以 `I` 开头，比如 `Recoverable`

通常它们会包含 0 个、最多 3 个方法（小接口，抽象程度高）

#### 空接口

> 如果一个类型 T 的方法集合是某接口类型 I 的方法集合的等价集合或超集，类型 T 实现了接口类型 I，
>
> 那么类型 T 的变量就可以作为合法的右值赋值给接口类型 I 的变量。

空接口类型的这一可接受任意类型变量值作为右值的特性，是 Go 加入泛型语法之前唯一一种具有“泛型”能力的语法元素

```go
var i interface{} = 15 // ok
i = "hello, golang" // ok
type T struct{}
var t T
i = t  // ok
i = &t // ok
```

#### 类型断言

一个接口类型的变量 `varI` 中可以包含任何类型的值，必须有一种方式来检测它的 **动态** 类型，即运行时在变量中存储的值的实际类型。

在执行过程中动态类型可能会有所不同，但是它总是可以分配给接口变量本身的类型。

类型 `T` 的值：

```go
v := varI.(T)       // unchecked type assertion，varI 必须是一个接口变量
if v, ok := varI.(T); ok {  // checked type assertion，一般用这种方法
    Process(v)
    return
}
// varI is not of type 
```

- 如果断言成功，变量 v 的类型为 i 的值的类型，而并非接口类型 T
- 如果断言失败，v 的类型信息为接口类型 T，它的值为 nil

#### 类型判断

接口变量的类型也可以使用一种特殊形式的 `switch` 来检测：**type-switch**

```go
switch t := areaIntf.(type) {
case *Square:
	fmt.Printf("Type Square %T with value %v\n", t, t)
case *Circle:
	fmt.Printf("Type Circle %T with value %v\n", t, t)
case nil:
	fmt.Printf("nil value: nothing to check?\n")
default:
	fmt.Printf("Unexpected type %T\n", t)
}
```

### Reflection

反射是用程序检查其所拥有的结构，尤其是类型的一种能力；这是元编程的一种形式。

反射可以在运行时检查类型和变量，例如它的大小、方法和 `动态` 的调用这些方法。

这对于没有源代码的包尤其有用，除非真得有必要，否则应当避免使用或小心使用。

变量的最基本信息就是类型和值：反射包的 `Type` 用来表示一个 Go 类型，反射包的 `Value` 为 Go 值提供了反射接口。

实际上，反射是通过检查一个接口的值，变量首先被转换成空接口。

```go
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value
```

更多查看

[反射包 unknwon/the-way-to-go_ZH_CN · GitHub](https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.10.md)

### Go 中的面向对象

OO 语言最重要的三个方面分别是：封装，继承和多态，在 Go 中它们是怎样表现的呢？

- 封装（数据隐藏）：和别的 OO 语言有 4 个或更多的访问层次相比，Go 把它简化为了 2 层:

  1）包范围内的：通过标识符首字母小写，`对象 ` 只在它所在的包内可见

  2）可导出的：通过标识符首字母大写，`对象 ` 对所在包以外也可见

类型只拥有自己所在包中定义的方法。

- 继承：用组合实现：内嵌一个（或多个）包含想要的行为（字段和方法）的类型；多重继承可以通过内嵌多个类型实现
- 多态：用接口实现：某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。Go 接口不是 Java 和 C# 接口的变体，而且接口间是不相关的，并且是大规模编程和可适应的演进型设计的关键。

## 并发

并发是一种能力，它让你的程序可以由若干个代码片段组合而成，并且每个片段都是独立运行的。

Go 并没有使用操作系统线程作为承载分解后的代码片段（模块）的基本执行单元，而是实现了**goroutine**这一由 Go 运行时（runtime）负责调度的、轻量的用户级线程，为并发程序设计提供原生支持。

### Goroutine

相比传统操作系统线程来说，goroutine 的优势主要是：

- 资源占用小，每个 goroutine 的初始栈大小仅为 2k
- 由 Go 运行时而不是操作系统调度，goroutine 上下文切换在用户层完成，开销更小
- 在语言层面而不是通过标准库提供。goroutine 由go 关键字创建，一退出就会被回收或销毁，开发体验更佳
- 语言内置 channel 作为 goroutine 间通信原语，为并发设计提供了强大支撑。

#### 创建

Go 语言通过`go关键字 +函数/方法 ` 的方式创建一个 goroutine。

创建后，新 goroutine 将拥有独立的代码执行流，并与创建它的 goroutine 一起被 Go 运行时调度。

```go
go fmt.Println("I am a goroutine")
var c = make(chan int)
go func(a, b int) { // 匿名函数/闭包
    c <- a + b
}(3,4) 

// $GOROOT/src/net/http/server.go
c := srv.newConn(rw)
go c.serve(connCtx) // 命名函数/方法
```

不需要考虑对 goroutine 的退出进行控制：**goroutine 的执行函数的返回，就意味着 goroutine 退出。**

如果需要手动控制，通过 `channel` 实现

### Channel

**传统语言的并发模型是基于对内存的共享的**

> 传统的编程语言（比如：C++、Java、Python 等）并非面向并发而生的，所以他们面对并发的逻辑多是基于操作系统的线程。
>
> 并发的执行单元（线程）之间的通信，利用的也是操作系统提供的线程或进程间通信的原语，
>
> 比如：共享内存、信号（signal）、管道（pipe）、消息队列、套接字（socket）等。

channel 既可以用来实现 Goroutine 间的通信，还可以实现 Goroutine 间的同步。

#### 创建

和切片、结构体、map 等一样，channel 也是一种复合数据类型。

channel 类型变量赋初值的唯一方法是使用 make 。

```go
var ch chan int // 声明 int 类型的 channel 类型变量，默认值为 nil

ch1 := make(chan int)  		//无缓冲
ch2 := make(chan int, 5) 	//带缓冲，5是缓冲长度

ch1 := make(chan<- int, 1) // 只发送channel类型
ch2 := make(<-chan int, 1) // 只接收channel类型
```

#### 发送和接收

Go 提供了 `<-` 操作符用于对 channel 类型变量进行发送与接收操作：

```go
ch1 <- 13    // 将整型字面值13发送到无缓冲channel类型变量ch1中
n := <- ch1  // 从无缓冲channel类型变量ch1中接收一个整型值存储到整型变量n中
ch2 <- 17    // 将整型字面值17发送到带缓冲channel类型变量ch2中
m := <- ch2  // 从带缓冲channel类型变量ch2中接收一个整型值存储到整型变量m中
```

- 无缓冲 channel：发送与接收操作同步，一定要放在两个不同的 Goroutine 中进行，否则会导致 deadlock
- 带缓冲 channel：发送或接收不需要阻塞等待，异步

```go
ch2 := make(chan int, 1)
n := <-ch2 // 由于此时ch2的缓冲区中无数据，因此对其进行接收操作将导致goroutine挂起

ch3 := make(chan int, 1)
ch3 <- 17  // 向ch3发送一个整型数17
ch3 <- 27  // 由于此时ch3中缓冲区已满，再向ch3发送数据也将导致goroutine挂起
```

#### 关闭

调用 go 内置的 `close()` 函数，**发送端负责关闭 channel**

```go
close(ch)


// 判断是否关闭可以用 c,ok
n := <- ch      // 当ch被关闭后，n将被赋值为ch元素类型的零值
m, ok := <-ch   // 当ch被关闭后，m将被赋值为ch元素类型的零值, ok值为false
for v := range ch { // 当ch被关闭后，for range循环结束
    ... ...
}
```

### Select

通过 select，我们可以同时在多个 channel 上进行发送 / 接收操作：

```go
select {
case x := <-ch1:     // 从channel ch1接收数据
  ... ...

case y, ok := <-ch2: // 从channel ch2接收数据，并根据ok值判断ch2是否已经关闭
  ... ...

case ch3 <- z:       // 将z值发送到channel ch3中:
  ... ...

default:             // 当上面case中的channel通信均无法实施时，执行该默认分支
}
```

channel 和 select 的结合使用能形成强大的表达能力：

- 利用 default 分支避免阻塞
- 实现超时机制
- 实现心跳机制

### Len

len 是 Go 语言的一个内置函数，它支持接收数组、切片、map、字符串和 channel 类型的参数，并返回对应类型的“长度”，也就是一个整型值。

针对 channel ch 的类型不同，len(ch) 有如下两种语义：

- 当 ch 为无缓冲 channel 时，len(ch) 总是返回 0
- 当 ch 为带缓冲 channel 时，len(ch) 返回当前 channel ch 中尚未被读取的元素个数

### 应用

#### 无缓冲

无缓冲 channel 兼具通信和同步特性，在并发程序中应用颇为广泛。

- 信号传递
- 替代锁机制

#### 带缓冲

带缓冲的 channel 与无缓冲的 channel 的最大不同之处，就在于它的**异步性。**

对一个带缓冲 channel：

- 在缓冲区未满的情况下，对它进行发送操作的 Goroutine 不会阻塞挂起
- 在缓冲区有数据的情况下，对它进行接收操作的 Goroutine 也不会阻塞挂起

所以可以用于：

- 消息队列
- 计数信号量

## 常用包

### Strings

作为一种基本数据结构，每种语言都有一些对于字符串的预定义处理函数。Go 中使用 `strings` 包来完成对字符串的主要操作。

[strings package - strings - pkg.go.dev](http://golang.org/pkg/strings/)

常用函数：

```go
// 判断前缀
strings.HasPrefix(s, prefix string) bool 
// 判断后缀
strings.HasSuffix(s, suffix string) bool
// 字符串包含
strings.Contains(s, substr string) bool
// 字符串替换
strings.Replace(str, old, new string, n int) string // 替换old的前 n 个字符， n=-1 替换整个 old
// 判断索引（出现位置）
strings.Index(s, str string) int // -1 表示不包含
strings.LastIndex(s, str string) int // 最后出现的位置
strings.IndexRune(s string, r rune) int // 非 ASCII 编码的字符，strings.IndexRune("chicken", 99)
// 出现次数
strings.Count(s, str string) int
// 重复字符串
strings.Repeat(s, count int) string //重复 count 次字符串 s 并返回一个新的字符串
// 大小写转换
strings.ToLower(s) string
strings.ToUpper(s) string
// 修剪
strings.TrimSpace(s) // 剔除开头和结尾的空白符号
strings.Trim(s, "cut") // 剔除开头和结尾指定字符，TrimLeft，TrimRight 只剔除一边
// 分割，返回 slice
strings.Fields(s) // 用一个或多个空白分割，全是空白则返回长度 0 的切片
strings.Split(s, sep) // 用指定字符串 seq 分割
// 拼接
strings.Join(sl []string, sep string) string // seq 为分隔符， sl 为字符串切片
// 例如：strings.Join([]string{"删除：文件夹", path}, " ")
```

### Strconv

与字符串相关的类型转换都是通过 `strconv` 包实现的。

[strconv package - strconv - pkg.go.dev](https://pkg.go.dev/strconv)

常用函数：

```go
strconv.IntSize // 当前操作系统 int 所占位置
// 整数转字符串
strconv.Itoa(i int) string
// 浮点数转字符串
strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string
	// 其中 fmt 表示格式（其值可以是 'b'、'e'、'f' 或 'g'）
	// prec 表示精度，bitSize 则使用 32 表示 float32，用 64 表示 float64
// 字符串转整数
strconv.Atoi(s string) (i int, err error)
// 浮点数转字符串
strconv.ParseFloat(s string, bitSize int) (f float64, err error)
```

### Time

`time` 包为我们提供了一个数据类型 `time.Time`（作为值使用）以及显示和测量时间和日期的功能函数。

[time package - time - pkg.go.dev](http://golang.org/pkg/time/)

常用函数：

```go
t := time.Now() // 当前时间
t.Day()
t.Minute()
// 标准格式化
t.Format("02 Jan 2006 15:04") // 输出 21 Jul 2011 10:31
```
