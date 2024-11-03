---
title: 代理模式
toc: true
categories: [System Design,Design Pattern]
tags: Patterns
date: 2022-06-30 10:23
updated: 2022-06-30 10:31
---
目标对象不可访问，通过代理对象增强访问。

- 控制目标对象的访问
- 增强功能

<!-- more -->

## 静态代理

具备以下特点：

- 目标对象和代理对象实现同一个业务接口
- 目标对象必须实现接口
- 代理对象在程序**运行前**就已经存在
- 能过灵活进行目标对象的切换，却无法进行功能的灵活处理

流程实现：

- `Service`：接口
- `Some implements Service`：目标对象
- `Agent implements Service`：代理对象
  - 在其中创建目标对象实现具体功能

> 面向接口编程：
>
> - 类中成员变量 设计为接口
> - 方法的形参 设计为接口
> - 方法的返回值 设计为接口
> - 调用时接口指向实现类

```java
/**
 * 静态代理拆分
 */
public class Agent implements Servcie{

    // 设计成员变量类型为接口，为了灵活切换目标对象
    public Servcie target;

    // 使用构造方法传入目标对象
    public Agent(Servcie target) {
        this.target = target;
    }

    @Override
    public void buy() {
        try {
            // 切面
            System.out.println("事务开启");
            // 业务
            target.buy();
            // 切面
            System.out.println("事务提交");
        } catch (Exception e) {
            System.out.println("事务回滚");
        }
    }
}

```

## 动态代理

代理对象在程序运行的过程中在内存构建，可以灵活的进行业务功能的切换。

### JDK 动态代理

- 目标对象必须实现业务接口
- 代理对象不必实现业务接口
- 动态代理的对象在程序运行前不存在，在运行时动态地在内存构建
- 动态代理能够灵活进行业务功能的切换
- 非接口中的方法不能被代理

```java
public class ProxyFactory {
    // 类中的成员变量接口
    Service target;
    // 传入目标对象
    public ProxyFactory(Service target){
    	this.target = target;
    }
    // 返回动态代理对象
    public static Object getAgent(AOP aop) {
        return Proxy.newProxyInstance(
                // 类加载器
                target.getClass().getClassLoader(),
                // 目标对象实现的所有接口
                target.getClass().getInterfaces(),
                // 代理功能实现
                new InvocationHandler() {
                    @Override
                    public Object invoke(
                      	Object proxy,	      // 代理对象 
                      	Method method, 	// 目标方法
                      	Object[] args		  // 目标方法的参数
                    ) throws Throwable {
                        Object obj = null;
                        try {
                            // 切面
                            aop.before();
                            // 业务
                            obj = method.invoke(target, args);
                            // 切面
                            aop.after();
                        } catch (Exception e) {
                            // 切面
                            aop.exception();
                        }
                        return obj;
                    }
                }
        );
    }
}
```

### CGLib 动态代理

又称子类代理，通过动态地在内存中构建子类对象，重写父类的方法进行代理功能的增强。

> 如果目标对象没有实现接口，则只能通过 CSLib 子类代理进行功能增强。

子类代理是对象字节码框架 ASM 实现，一般通过 Spring 这些框架直接使用。
