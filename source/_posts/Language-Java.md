---
title: Java
toc: true
categories: [Programming Language,Java]
tags: Java
date: 2022-07-09 20:26
updated: 2022-07-09 20:31
references:
  - title: JavaGuide
    url: https://javaguide.cn/
  - title: Java核心技术卷
    url: https://book.douban.com/subject/34898994/
---

Write Once, Run Anywhere!

一些 Java 学习笔记。

<!-- more -->

## 概念

### JVM Vs JDK Vs JRE

- JVM（Java Virtual Machine）：运行 Java 字节码的虚拟机
- JDK（Java Development Kit）： 功能齐全的 Java 开发工具，包含 JRE、编译器（Javac）和工具（如 javadoc 和 jdb）
- JRE（Java Runtime Environment）：Java 运行时环境，包括 JVM、Java 类库、Java 命令和其他的一些基础构件

### 字节码文件 .class

JVM 可以理解的代码就叫做字节码（即扩展名为 `.class` 的文件），它不面向任何特定的处理器，只面向虚拟机。

![Java程序转变为机器代码的过程](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202207072159233.png)

> **为什么说 Java 语言“编译与解释并存”？**
>
> Java 语言既具有编译型语言的特征，也具有解释型语言的特征。
>
> 1. 由 Java 编写的程序需要先经过编译步骤，生成字节码（`.class` 文件）
> 2. 这种字节码必须由 Java 解释器来解释执行。

### 三大特征

#### 封装

封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。

#### 继承

继承是使用已存在的类的定义作为基础建立新类的基础，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。

#### 多态

多态表示一个对象具有多种的状态，具体表现为父类的引用指向子类的实例。

**多态的特点:**

- 对象类型和引用类型之间具有继承（类）/实现（接口）的关系；
- 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定；
- 多态不能调用“只在子类存在但在父类不存在”的方法；
- 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法。

## 基础

### 静态 Static

- **静态域**（静态变量）：如果将域定义为 `static`, 每个类中只有一个这样的域。也就是所有对象共享同一个静态域，不需要拷贝。
- **静态常量**：相较于静态域，使用的更加频繁，用 `static final` 修饰。
- **静态方法**：特性同上，与普通方法的区别是不能向对象实施操作，即不能调用 `this`，但是可以调用类中的静态域。**调用静态方法可以无需创建对象**，一般使用 ` 类名.方法名` 的方式来调用静态方法。

#### 静态方法为什么不能调用非静态成员

 1. 静态方法是属于类的，在类加载的时候就会分配内存，可以通过类名直接访问。而非静态成员属于实例对象，只有在对象实例化之后才存在，需要通过类的实例对象去访问。
 2. 在类的非静态成员不存在的时候静态成员就已经存在了，此时调用在内存中还不存在的非静态成员，属于非法操作。

### 重载 & 重写

- 重载：多个方法有相同的名字、不同的参数。同样的一个方法能够根据输入数据的不同，做出不同的处理。
- 重写：子类继承自父类的相同方法，`@Override`

### 可变长参数

从 Java5 开始，Java 支持定义可变长参数，即允许在调用方法时传入不定长度的参数。

可变参数只能作为函数的最后一个参数，但其前面可以有也可以没有任何其他参数。

**方法重载会优先匹配固定参数**

```java
public static void method(String arg1, String... args) {
   //......
}
```

### 基本类型 & 包装类

- 基本类型：
  - 6 种数字类型：
	- 4 种整数型：`byte`、`short`、`int`、`long`（数值后面加 L）
	- 2 种浮点型：`float`、`double`
  - 1 种字符类型：`char`
  - 1 种布尔型：`boolean`
- 包装类：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Character`、`Boolean`

**所有整型包装类对象之间值的比较，全部使用 equals 方法比较**

#### 基本类型和包装类型的区别

 - 成员变量包装类型不赋值就是 null ，而基本类型有默认值且不是 null。
 - 包装类型可用于泛型，而基本类型不可以。
 - 基本数据类型的局部变量存放在 Java 虚拟机栈中的局部变量表中，基本数据类型的成员变量（未被 static 修饰 ）存放在 Java 虚拟机的堆中。包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中。
 - 相比于对象类型， 基本数据类型占用的空间非常小。

#### 包装类型的缓存机制

 Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能:

 - Byte,Short,Integer,Long 这 4 种包装类默认创建了数值 [-128，127] 的相应类型的缓存数据
 - Character 创建了数值在 [0,127] 范围的缓存数据
 - Boolean 直接返回 True or False

#### 装箱 & 拆箱

- **装箱**：将基本类型用它们对应的引用类型包装起来
- **拆箱**：将包装类型转换为基本数据类型

```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```

**尽量避免不必要的拆装箱操作**，频繁拆装箱会严重影响系统的性能。

## 对象和类

### 抽象类 & 接口

抽象类除了不能实例化对象之外，类的其它功能和普通类一样。

由于抽象类不能实例化对象，所以抽象类必须被继承，才能被使用。也是因为这个原因，通常在设计阶段决定要不要设计抽象类。

父类包含了子类集合的常见的方法，但是由于父类本身是抽象的，所以不能使用这些方法。

在 Java 中抽象类表示的是一种继承关系，一个类只能继承一个抽象类，而一个类却可以实现多个接口。

#### 抽象类和接口的区别

 - 共同点：接口和抽象类都是继承树的上层
   
   - 都是上层的抽象层
   
   - 都不能被实例化
   - 都能包含抽象的方法，这些抽象的方法用于描述类具备的功能，但是不比提供具体的实现
   - 都可以有默认实现的方法（Java 8 可以用 `default` 关键字在接口中定义默认方法）
   
 - 区别：
   - 抽象类中可有非抽象的方法，可以代码复用；接口只能有抽象方法
   - 一个类只能继承一个类，但是可以实现多个接口
   - 接口中的成员变量只能是 `public static final` 类型的，不能被修改且必须有初始值，而抽象类的成员变量默认 default，可在子类中被重新定义，也可被重新赋值。

#### 访问修饰符

- 仅对本类可见 private
- 对所有类可见 public
 - 对本包和所有子类可见 protected
 - 对本包可见—默认 不需要修饰符

### 深拷贝 & 浅拷贝

- **引用拷贝**：两个不同的引用指向同一个对象。
- **浅拷贝**：浅拷贝会在堆上创建一个新的对象（区别于引用拷贝的一点），不过，如果原对象内部的属性是引用类型的话，浅拷贝会直接复制内部对象的引用地址，也就是说**拷贝对象和原对象共用同一个内部对象**。
- **深拷贝** ：深拷贝会**完全复制整个对象**，包括这个对象所包含的内部对象。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202207091112775.png" />

```java
// 浅拷贝
@Override
public Address clone() {
	try {
		return (Address) super.clone();
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
// 深拷贝
@Override
public Person clone() {
    try {
        Person person = (Person) super.clone(); // 拷贝对象
        person.setAddress(person.getAddress().clone());
        return person;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}

```

### Object: 所有类的超类

可以使用 Object 类型的变量引用任何类型的对象：

`Object obj = new Employee("Harry Hacker", 35000);`

要想对其中的内容进行具体的操作， 还需要清楚对象的原始类型， 并进行相应的类型转换：

`Employee e = (Employee) obj;`

#### hashcode()

- 如果两个对象的 `hashCode` 值相等，那这两个对象不一定相等（哈希碰撞）。
- 如果两个对象的 `hashCode` 值相等并且 `equals()` 方法也返回 `true`，我们才认为这两个对象相等。
- 如果两个对象的 `hashCode` 值不相等，我们就可以直接认为这两个对象不相等。

### String

- `String` 中的对象是不可变的，也就可以理解为常量，线程安全。
- `StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。
- `StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

**对于三者使用的总结：**

1. 操作少量的数据: 适用 `String`
2. 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
3. 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`

## 范型

编译器可以对泛型参数进行检测，并且通过泛型参数可以指定传入的对象类型。

### 范型类

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}

Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

### 范型接口

```java
public interface Generator<T> {
    public T method();
}

// 不指定类型实现接口
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}

// 指定类型实现接口
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}
```

### 范型方法

```java
   public static < E > void printArray( E[] inputArray ) {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }

// 创建不同类型数组： Integer, Double 和 Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```
