---
title: Spring
toc: true
categories: [Framework,Spring]
tags: 
	- SSM
date: 2022-07-05 09:08
updated: 2022-07-05 09:16
---

Spring 是一个**容器**。

Spring 框架的目标是使 J2EE 开发变得更容易使用，通过启用基于 POJO 的编程模型来促进良好的编程实践。

<!-- more -->

## 介绍

### 特点

- 轻量级：20 多个模块，每隔 jar 包基本小于 1M，核心包 3M 左右
- 面向接口编程：可拓展、可维护
- AOP：面向切面编程，底层原理是动态代理。合并公共代码，单独开发
- 整合其他框架

### 核心

- 控制反转（IoC）：“解耦合”，通过依赖注入 DI（Dependency Injection）实现
- 面向切面编程（AOP）：“注入”，下面通过 AspectJ 实现

## 三层架构

### 非 Spring

- 实体类
- 数据访问层 dao
  
  - Mapper.java（接口） 
  
  - MapperImpl.java（实现类）
- 业务逻辑层
  
  - Service.java（接口）
  
  - ServiceImpl.java（实现类）
- 界面层 controller

### Spring 接管

不 new 对象，使用 IoC 在 `applicationContext.xml` 中创建，注意首先需要空构造方法和 `setXXX()`。

## IoC

IoC 是一种思想，由 Spring 容器进行对象创建和依赖注入，程序员直接使用。

- 正转：程序员对象创建和依赖注入

  `Student stu = new Student()`

- 反转：Spring 容器对象创建和依赖注入

  `<bean id="stu" class="…"></bean>`

### 基于 XML

- 创建对象：`<bean id="stu" class="…"></bean>`
- 对象赋值：
  - setter 注入：必须提供无参构造方法，必须提供 setXXX()
	- 简单类型注入：value 属性 `<property name="age" value="11"/>`
	- 引用类型注入：ref 属性 `<property name="school" ref="sch"/>`
  - 构造方法注入：
	- 使用构造方法的参数名称：`<constructor-arg name="school" value="Hohai"/>`
	- 使用构造方法的参数下标：`<constructor-arg index="0" value="Hohai"/>`
	- 使用默认构造方法的参数顺序：`<constructor-arg value="Hohai"/>`

### 基于注解的 IoC

也称 DI，是 IoC 是具体实现的技术。即从 xml 换为注解 annotation。

- 创建对象的注解
  - `@Component`：创建任意对象
  - `@Controller`：创建控制器对象，接收用户请求，返回结果给客户端
  - `@Service`：创建业务逻辑层对象，负责向下访问数据访问层，处理完毕返回结果给界面层
  - `@Respository`：创建数据访问层对象，负责数据库增删改查操作
- 依赖注入的注解
  - 值类型（8 种基本类型和 String）的注入
	- `@Value`：给简单类型注入值
  - 引用类型的注入
	- `@Autowired`：使用类型注入值，从整个 Bean 工厂搜索同源类型的对象进行注入
	  - 同源：被注入的类型（Student 中的 School）与注入的类型**完全相同/父子/接口和实现类**的类型
	  - 父子类用下面的注解区别
	- `@Qualifier`：使用名称注入值，从整个 Bean 工厂搜索相同名称的对象进行注入
	  - 需要上面那个注解，否则无法注入

  **要在 Spring 的核心配置文件中添加包扫描：**`<context:component-scan base-package="top.sukiu.annotation"/>`

  - 单个包扫描（推荐）：一个个包扫
  - 多个包扫描：以逗号/空格/分号分隔
  - 扫描根包（不推荐）

## AspectJ

一个优秀的面向切面的框架，拓展 Java 语言，提供强大的切面实现。

**切入点表达式：**`execution(访问权限 方法返回值 方法声明（参数）异常类型)`，简化后 `execution(方法返回值 方法声明（参数）`

- `*`：任意字符
- `..`：出现在
  - 方法声明的参数中：任意参数
  - 路径中：本路径及其所有子路径

> 例子：
>
> `execution(public * *(..))`：任意的公共方法
>
> `execution(* set*(..))`：任意以 set 开头的方法
>
> `execution(* com.xyz.service.impl.*.*(..))`：任意返回值类型，在此包任意类型的任意方法的任意参数

常用的通知类型：

- 前置通知 `@Before`：在目标方法执行前切入切面功能。
	```java
    /*
    * 前置通知规范：
    * 1）权限public
    * 2）返回值void
    * 3）方法名自定义
    * 4）方法无参数，有只能是JoinPoint类型（包含目标方法的签名和参数）
    */
    @Before(value = "execution(public * com.aspect.SomeServiceImpl.*(..))")
    public void myBefore() {
    	System.out.println("前置功能切入");
    }
  ```
- 后置通知 `@AfterReturning`：在目标方法执行后切入切面功能。
	```java
    /*
    * 后置通知规范：
    * 1）权限public
    * 2）返回值void
    * 3）方法名自定义
    * 4）方法有/无参数（一般有）
    */
    @AfterReturning(value = "execution(public * com.aspect.SomeServiceImpl.*(..))", returning = "obj")
    public void myAfterReturning(Object obj) {
    	System.out.println("后置功能切入");
    }
    // 基本类型和String的返回值无法改变
  ```
- 环绕通知 `@Around`：通过拦截目标方法，在前后增强功能的通知，功能强大，一般事务使用。
	```java
    /*
    * 环绕通知规范：
    * 1）权限public
    * 2）返回值 = 目标方法返回值
    * 3）方法名自定义
    * 4）方法有参数 = 目标方法
    */
     @Around(value = "execution(public * com.aspect.SomeServiceImpl.*(..))")
    public String myAround(ProceedingJoinPoint p) throws Throwable{
    	// 前置
        System.out.println("Before");
        // 目标方法
        Object obj = p.proceed(p.getArgs());
        // 后置
        System.out.println("After");
        return obj.toString().toUpperCase();
    }
    // 可以修改返回值
  ```
- 最终通知 `@After`：无论目标方法是否正常执行，最终通知都会执行。
	```java
    /*
    * 最终通知规范：
    * 1）权限public
    * 2）返回值void
    * 3）方法名自定义
    * 4）方法无参数，有只能是JoinPoint类型（包含目标方法的签名和参数）
    */
    @After(value = "execution(public * com.aspect.SomeServiceImpl.*(..))")
    public void myAfter() {
    	System.out.println("最终功能切入");
    }
  ```
- 定义切入点 `@Pointcut`：多个切面切入同一个切入点，起别名
	```java
    @Pointcut(value = "execution(public * com.aspect.SomeServiceImpl.*(..))")
    public void myCut(){}
    
    @Before(value = "myCut()")
    public void myBefore() {
    	System.out.println("前置功能切入");
    }
  ```

### JDK/CGLib 动态代理

`applicationContext.xml` 中：

- `<aop:aspectj-autoproxy/>`：默认 JDK 动态代理，必须使用接口类型
- `<aop:aspectj-autoproxy proxy-target-class="true"/>`：切换 CGLib 子类代理，可以用接口或者实现类

> **一直用接口即可！！！**
