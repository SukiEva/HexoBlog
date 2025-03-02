---
title: 什么是POJO？
toc: true
categories: [Programming Language,Java]
tags: Java
date: 2022-07-12 21:29:53
updated: 2022-07-12 21:56
references:
  - title: 什么是JavaBean、bean? 什么是POJO、PO、DTO、VO、BO ? 什么是EJB、EntityBean？
    url: https://www.cnblogs.com/panchanggui/p/11610998.html
  - title: java的(PO,VO,TO,BO,DAO,POJO)类名包名解释
    url: https://www.cnblogs.com/549294286/p/4463932.html
---

在划分项目结构时，对实体类划分的各种命名。

- POJO：简单的 Java 对象
- VO：值对象、视图对象
- PO：持久对象
- QO：查询对象
- DAO：数据访问对象——同时还有 DAO 模式
- DTO：数据传输对象——同时还有 DTO 模式

<!-- more -->

## POJO

POJO（Plain Ordinary Java Object）简单的 Java 对象，实际就是普通 JavaBeans，是为了避免和 EJB（Enterprise JavaBean）混淆所创造的简称。

具有一部分 getter/setter 方法的那种类就可以称作 POJO，POJO 不担当任何的特殊的角色，也不实现任何 Java 框架指定的接口。

```java
public class DbHello { //简单的Java类，称之为POJO，不继承，不实现接口

   　　private DictionaryDAO dao;
   　　public void setDao(DictionaryDAO dao) {
          　　this.dao = dao;
   　　}

}
```

> POJO 可以认为是一个中间对象：
>
> 一个中间对象，可以转化为 PO、DTO、VO。
>
> 1 ．POJO 持久化之后 -> PO（在运行期，由 Hibernate 中的 cglib 动态把 POJO 转换为 PO，PO 相对于 POJO 会增加一些用来管理数据库 entity 状态的属性和方法。PO 对于 programmer 来说完全透明，由于是运行期生成 PO，所以可以支持增量编译，增量调试）
>
> 2 ．POJO 传输过程中 -> DTO
>
> 3 ．POJO 用作表示层 -> VO

## PO

PO 全称是 persistant object （持久对象）。

最形象的理解就是一个 PO 就是数据库中的一条记录。好处是可以把一条记录作为一个对象处理，可以方便的转为其它对象。

```java
public class User {

    private long id;
    private String name;

    public void setId(long id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

} 
```

## VO

VO 全称是 value object 值对象 / view object 表现层（视图）对象。

- Value Object：通常用于业务层之间的数据传递，和 PO 一样也是仅仅包含数据而已。
- View Object：用一个 VO 对象对应整个界面的值，如对于一个 WEB 页面，或者 SWT、SWING 的一个界面，。

## DTO

DTO （TO） 全称是 Data Transfer Object （数据传输对象），主要用于远程调用等需要大量传输对象的地方。

将 PO 中的部分属性抽取出来，就形成了 DTO。

> 比如我们一张表有 100 个字段，那么对应的 PO 就有 100 个属性。
>
> 但是我们界面上只要显示 10 个字段，客户端用 WEB service 来获取数据，没有必要把整个 PO 对象传递到客户端，这时我们就可以用只有这 10 个属性的 DTO 来传递结果到客户端，这样也不会暴露服务端表结构.到达客户端以后，如果用这个对象来对应界面显示，那此时它的身份就转为 VO。

## DAO

DAO 全称是 data access object （数据访问对象），主要用来封装对数据库的访问。

通常和 PO 结合使用，DAO 中包含了各种数据库的操作方法。它可以把 POJO 持久化为 PO，用 PO 组装出来 VO、DTO
