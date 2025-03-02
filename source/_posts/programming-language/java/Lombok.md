---
title: Lombok
toc: true
categories: [Programming Language,Java]
tags: Annotation
date: 2022-10-16 16:49
updated: 2022-10-16 18:18
references:
	- title: Project Lombok
	  url: https://projectlombok.org/features/
---

lombok 通过注解消除实际开发中的样板式代码：getter、setter 方法，重写 toString、equals 方法等。

<!-- more -->

## 注解

### @NonNull

[@NonNull](https://projectlombok.org/features/NonNull)

@NonNull 注解可以放在方法，参数或字段上。

该注解不是断言非空，而是**在赋值该字段时对参数进行判空检查**，如果已经有了 `if (xxx == null)` 则不会有其他操作。

**With Lombok**

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

**Vanilla Java**

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    if (person == null) {
      throw new NullPointerException("person is marked non-null but is null");
    }
    this.name = person.getName();
  }
}
```

### @Cleanup

[@Cleanup](https://projectlombok.org/features/Cleanup)

自动调用 `close()` 方法，如果还需要做异常处理，明显不如 `try-with` 方便。

**With Lombok**

```java

import lombok.Cleanup;
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    @Cleanup InputStream in = new FileInputStream(args[0]);
    @Cleanup OutputStream out = new FileOutputStream(args[1]);
    byte[] b = new byte[10000];
    while (true) {
      int r = in.read(b);
      if (r == -1) break;
      out.write(b, 0, r);
    }
  }
}
```

**Vanilla Java**

```java
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    InputStream in = new FileInputStream(args[0]);
    try {
      OutputStream out = new FileOutputStream(args[1]);
      try {
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      } finally {
        if (out != null) {
          out.close();
        }
      }
    } finally {
      if (in != null) {
        in.close();
      }
    }
  }
}
```

### @Getter/@Setter

[@Getter and @Setter](https://projectlombok.org/features/GetterSetter)

自动生成属性的 get/set 方法，默认都是 public，可通过 AccessLevel 设置可见性。

- 作用类上，生成所有非静态成员变量的 getter/setter 方法
- 作用于成员变量上，生成该成员变量的 getter/setter 方法
- Getter(lazy=true) 懒加载

**With Lombok**

```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;

public class GetterSetterExample {
  @Getter @Setter private int age = 10;
  
  @Setter(AccessLevel.PROTECTED) private String name;
  
  @Override 
  public String toString() {
    return String.format("%s (age: %d)", name, age);
  }
}
```

**Vanilla Java**

```java
public class GetterSetterExample {
  private int age = 10;

  private String name;
  
  @Override 
  public String toString() {
    return String.format("%s (age: %d)", name, age);
  }
  
  public int getAge() {
    return age;
  }
  
  public void setAge(int age) {
    this.age = age;
  }
  
  protected void setName(String name) {
    this.name = name;
  }
}
```

### @ToString

[@ToString](https://projectlombok.org/features/ToString)

作用于类，覆盖默认的 toString() 方法。

- 通过 includeFieldNames 属性限定显示某些字段
- 通过 exclude 属性排除某些字段
- callSuper 是否输出超类

**With Lombok**

```java
import lombok.ToString;

@ToString
public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  @ToString.Exclude private int id;
  
  public String getName() {
    return this.name;
  }
  
  @ToString(callSuper=true, includeFieldNames=true)
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

**Vanilla Java**

```java

import java.util.Arrays;

public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;
  
  public String getName() {
    return this.name;
  }
  
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
    
    @Override public String toString() {
      return "Square(super=" + super.toString() + ", width=" + this.width + ", height=" + this.height + ")";
    }
  }
  
  @Override public String toString() {
    return "ToStringExample(" + this.getName() + ", " + this.shape + ", " + Arrays.deepToString(this.tags) + ")";
  }
}
```

### @EqualsAndHashCode

[@EqualsAndHashCode](https://projectlombok.org/features/EqualsAndHashCode)

作用于类，覆盖默认的 equals 和 hashCode。

- `doNotUseGetters` = [`true` | `false`] (default: false)
- `callSuper` = [`call` | `skip` | `warn`] (default: warn)
- `flagUsage` = [`warning` | `error`] (default: not set)

**With Lombok**

```java
import lombok.EqualsAndHashCode;

@EqualsAndHashCode
public class EqualsAndHashCodeExample {
  private transient int transientVar = 10;
  private String name;
  private double score;
  @EqualsAndHashCode.Exclude private Shape shape = new Square(5, 10);
  private String[] tags;
  @EqualsAndHashCode.Exclude private int id;
  
  public String getName() {
    return this.name;
  }
  
  @EqualsAndHashCode(callSuper=true)
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

**Vanilla Java**

```java
import java.util.Arrays;

public class EqualsAndHashCodeExample {
  private transient int transientVar = 10;
  private String name;
  private double score;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;
  
  public String getName() {
    return this.name;
  }
  
  @Override 
  public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof EqualsAndHashCodeExample)) return false;
    EqualsAndHashCodeExample other = (EqualsAndHashCodeExample) o;
    if (!other.canEqual((Object)this)) return false;
    if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
    if (Double.compare(this.score, other.score) != 0) return false;
    if (!Arrays.deepEquals(this.tags, other.tags)) return false;
    return true;
  }
  
  @Override 
  public int hashCode() {
    final int PRIME = 59;
    int result = 1;
    final long temp1 = Double.doubleToLongBits(this.score);
    result = (result*PRIME) + (this.name == null ? 43 : this.name.hashCode());
    result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
    result = (result*PRIME) + Arrays.deepHashCode(this.tags);
    return result;
  }
  
  protected boolean canEqual(Object other) {
    return other instanceof EqualsAndHashCodeExample;
  }
  
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
    
    @Override 
    public boolean equals(Object o) {
      if (o == this) return true;
      if (!(o instanceof Square)) return false;
      Square other = (Square) o;
      if (!other.canEqual((Object)this)) return false;
      if (!super.equals(o)) return false;
      if (this.width != other.width) return false;
      if (this.height != other.height) return false;
      return true;
    }
    
    @Override 
    public int hashCode() {
      final int PRIME = 59;
      int result = 1;
      result = (result*PRIME) + super.hashCode();
      result = (result*PRIME) + this.width;
      result = (result*PRIME) + this.height;
      return result;
    }
    
    protected boolean canEqual(Object other) {
      return other instanceof Square;
    }
  }
}
```

### @NoArgsConstructor, @RequiredArgsConstructor And @AllArgsConstructor

[@NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor](https://projectlombok.org/features/constructor)

生成各种构造函数，看名字知用途，其中 `@RequiredArgsConstructor ` 通过 `@NonNull` 获取需要构造的属性。

**With Lombok**

```java
import lombok.AccessLevel;
import lombok.RequiredArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.NonNull;

@RequiredArgsConstructor(staticName = "of")
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class ConstructorExample<T> {
  private int x, y;
  @NonNull private T description;
  
  @NoArgsConstructor
  public static class NoArgsExample {
    @NonNull private String field;
  }
}
```

**Vanilla Java**

```java
public class ConstructorExample<T> {
  private int x, y;
  @NonNull private T description;
  
  private ConstructorExample(T description) {
    if (description == null) throw new NullPointerException("description");
    this.description = description;
  }
  
  public static <T> ConstructorExample<T> of(T description) {
    return new ConstructorExample<T>(description);
  }
  
  @java.beans.ConstructorProperties({"x", "y", "description"})
  protected ConstructorExample(int x, int y, T description) {
    if (description == null) throw new NullPointerException("description");
    this.x = x;
    this.y = y;
    this.description = description;
  }
  
  public static class NoArgsExample {
    @NonNull private String field;
    
    public NoArgsExample() {
    }
  }
}
```

### @Data

[@Data](https://projectlombok.org/features/Data)

作用于类上，是以下注解的集合：

`@ToString @EqualsAndHashCode @Getter @Setter @RequiredArgsConstructor`

### @Value

[@Value](https://projectlombok.org/features/Value)

用于一个不可变的对象类，区别于 `@Data` 在于没有 `@Setter`，属性默认 `final`，构造函数是 `@AllArgsConstructor`。

### @Builder

[@Builder](https://projectlombok.org/features/Builder)

作用于类上，将类转变为建造者模式。

**With Lombok**

```java
import lombok.Builder;
import lombok.Singular;
import java.util.Set;

@Builder
public class BuilderExample {
  @Builder.Default private long created = System.currentTimeMillis();
  private String name;
  private int age;
  @Singular private Set<String> occupations;
}
```

**Vanilla Java**

```java
import java.util.Set;

public class BuilderExample {
  private long created;
  private String name;
  private int age;
  private Set<String> occupations;
  
  BuilderExample(String name, int age, Set<String> occupations) {
    this.name = name;
    this.age = age;
    this.occupations = occupations;
  }
  
  private static long $default$created() {
    return System.currentTimeMillis();
  }
  
  public static BuilderExampleBuilder builder() {
    return new BuilderExampleBuilder();
  }
  
  public static class BuilderExampleBuilder {
    private long created;
    private boolean created$set;
    private String name;
    private int age;
    private java.util.ArrayList<String> occupations;
    
    BuilderExampleBuilder() {
    }
    
    public BuilderExampleBuilder created(long created) {
      this.created = created;
      this.created$set = true;
      return this;
    }
    
    public BuilderExampleBuilder name(String name) {
      this.name = name;
      return this;
    }
    
    public BuilderExampleBuilder age(int age) {
      this.age = age;
      return this;
    }
    
    public BuilderExampleBuilder occupation(String occupation) {
      if (this.occupations == null) {
        this.occupations = new java.util.ArrayList<String>();
      }
      
      this.occupations.add(occupation);
      return this;
    }
    
    public BuilderExampleBuilder occupations(Collection<? extends String> occupations) {
      if (this.occupations == null) {
        this.occupations = new java.util.ArrayList<String>();
      }

      this.occupations.addAll(occupations);
      return this;
    }
    
    public BuilderExampleBuilder clearOccupations() {
      if (this.occupations != null) {
        this.occupations.clear();
      }
      
      return this;
    }

    public BuilderExample build() {
      // complicated switch statement to produce a compact properly sized immutable set omitted.
      Set<String> occupations = ...;
      return new BuilderExample(created$set ? created : BuilderExample.$default$created(), name, age, occupations);
    }
    
    @java.lang.Override
    public String toString() {
      return "BuilderExample.BuilderExampleBuilder(created = " + this.created + ", name = " + this.name + ", age = " + this.age + ", occupations = " + this.occupations + ")";
    }
  }
}
```

### @SneakyThrows

[@SneakyThrows](https://projectlombok.org/features/SneakyThrows)

可以对受检异常进行捕捉并抛出，根据官方介绍大概抛出之前没检查的异常。

> To boldly throw checked exceptions where no one has thrown them before!

**With Lombok**

```java
import lombok.SneakyThrows;

public class SneakyThrowsExample implements Runnable {
  @SneakyThrows(UnsupportedEncodingException.class)
  public String utf8ToString(byte[] bytes) {
    return new String(bytes, "UTF-8");
  }
  
  @SneakyThrows
  public void run() {
    throw new Throwable();
  }
}
```

**Vanilla Java**

```java
import lombok.Lombok;

public class SneakyThrowsExample implements Runnable {
  public String utf8ToString(byte[] bytes) {
    try {
      return new String(bytes, "UTF-8");
    } catch (UnsupportedEncodingException e) {
      throw Lombok.sneakyThrow(e);
    }
  }
  
  public void run() {
    try {
      throw new Throwable();
    } catch (Throwable t) {
      throw Lombok.sneakyThrow(t);
    }
  }
}
```

### @Synchronized

[@Synchronized](https://projectlombok.org/features/Synchronized)

作用于方法级别，可以替换 synchronize 关键字或 lock 锁，用处不大。

**With Lombok**

```java
 
import lombok.Synchronized;

public class SynchronizedExample {
  private final Object readLock = new Object();
  
  @Synchronized
  public static void hello() {
    System.out.println("world");
  }
  
  @Synchronized
  public int answerToLife() {
    return 42;
  }
  
  @Synchronized("readLock")
  public void foo() {
    System.out.println("bar");
  }
}
```

**Vanilla Java**

```java
 public class SynchronizedExample {
  private static final Object $LOCK = new Object[0];
  private final Object $lock = new Object[0];
  private final Object readLock = new Object();
  
  public static void hello() {
    synchronized($LOCK) {
      System.out.println("world");
    }
  }
  
  public int answerToLife() {
    synchronized($lock) {
      return 42;
    }
  }
  
  public void foo() {
    synchronized(readLock) {
      System.out.println("bar");
    }
  }
}
```

### @With

[@With](https://projectlombok.org/features/With)

克隆对象，修改一个值而保留其他值不变。

**With Lombok**

```java
 import lombok.AccessLevel;
import lombok.NonNull;
import lombok.With;

public class WithExample {
  @With(AccessLevel.PROTECTED) @NonNull private final String name;
  @With private final int age;
  
  public WithExample(@NonNull String name, int age) {
    this.name = name;
    this.age = age;
  }
}
```

**Vanilla Java**

```java
 import lombok.NonNull;

public class WithExample {
  private @NonNull final String name;
  private final int age;

  public WithExample(String name, int age) {
    if (name == null) throw new NullPointerException();
    this.name = name;
    this.age = age;
  }

  protected WithExample withName(@NonNull String name) {
    if (name == null) throw new java.lang.NullPointerException("name");
    return this.name == name ? this : new WithExample(name, age);
  }

  public WithExample withAge(int age) {
    return this.age == age ? this : new WithExample(name, age);
  }
}
```

### @Log

[@Log (and friends)](https://projectlombok.org/features/log)

作用于类上，生成日志变量。

针对不同的日志实现产品，有不同的注解。

```java
@Log4j
//=
private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);

@Slf4j
//=
private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
```

> SLF4J (Simple logging Facade for Java) 不是一个真正的日志实现，而是一个**抽象层**（ abstraction layer），它**允许你在后台使用任意一个日志类库**。

## 语法糖

### Val

[val](https://projectlombok.org/features/val)

用在局部变量前面，声明为 final 常量。

### Var

[var](https://projectlombok.org/features/var)

用在局部变量前面，声明变量。

## 实验性

更多开发中的功能详见：

[Experimental](https://projectlombok.org/features/experimental/)
