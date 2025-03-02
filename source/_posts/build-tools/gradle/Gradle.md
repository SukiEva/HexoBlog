---
title: Gradle
toc: true
categories: [Build Tools, Gradle]
tags: Gradle
date: 2022-07-15 10:20
updated: 2022-07-15 11:30
references:
  - title: Gradle 官方文档
    url: https://docs.gradle.org/current/userguide/getting_started.html
  - title: Gradle User Guide 中文版
    url: https://doc.yonyoucloud.com/doc/wiki/project/GradleUserGuide-Wiki/index.html
---

Gradle 是一种开源构建自动化的工具，旨在灵活地构建几乎任何类型的软件。

<!-- more -->

## 概览

Gradle 是一种开源构建自动化的工具，旨在灵活地构建几乎任何类型的软件。

- 高性能：避免重复任务、缓存复用
- 基于 JVM：需要 JDK 环境、可用 Java API、跨平台、也适用于其他原生项目
- 约定：借鉴 Maven 经验、可用各种插件、可自定义插件和任务
- 可拓展性：可自定义任务类型和构建模型（比如 Android）
- IDE 支持：Android Studio, IntelliJ IDEA, Eclipse, and NetBeans.
- 检视：构建时提供输出信息供查错

更多特点参考：

{% link Gradle 中文文档::https://doc.yonyoucloud.com/doc/wiki/project/GradleUserGuide-Wiki/overview/features.html %}

## 基础

Gradle 基于下面两个基础概念:

- projects (项目)：每一个 project 是由一个或多个 tasks 构成的
- tasks (任务)：更加细化的构建，可能是编译一些 classes、创建一个 JAR、生成 javadoc 等

### 任务定义

```groovy
task hello << {
    println 'Hello world!'
}

task upper << {
    String someString = 'mY_nAmE'
    println "Original: " + someString
    println "Upper case: " + someString.toUpperCase()
}
```

通过 `gradle -q hello` 运行

> `-q`：不会生成 Gradle 的日志信息 (log messages), 所以用户只能看到 tasks 的输出

### 任务依赖

执行 `gradle -q intro` 时，会先执行 `hello` 任务：

```groovy
task hello << {
    println 'Hello world!'
}

task intro(dependsOn: hello) << {
    println "I'm Gradle"
}
```

> 在加入一个依赖之前，这个依赖的任务不需要提前定义，即顺序无所谓。

### 动态任务

动态创建 task1, task2, task3, task4：

```groovy
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}
```

输出：

```shell
$ gradle -q task1
I'm task number 1
```

### 使用已有任务

#### 使用依赖

使用上面动态任务的例子：

```groovy
task0.dependsOn task2, task3
```

输出：

```shell
$ gradle -q task0
I'm task number 2
I'm task number 3
I'm task number 0
```

#### 使用行为

`doFirst` 和 `doLast` 可以被执行许多次，`<<` 是 `doLast` 简写：

```groovy
task hello << {
    println 'Hello Earth'
}
hello.doFirst {
    println 'Hello Venus'
}
hello.doLast {
    println 'Hello Mars'
}
hello << {
    println 'Hello Jupiter'
}
```

输出：

```shell
$ gradle -q hello
Hello Venus
Hello Earth
Hello Mars
Hello Jupiter
```

### 短标记

短标记 `$` 可以访问一个存在的任务：

```groovy
task hello << {
    println 'Hello world!'
}
hello.doLast {
    println "Greetings from the $hello.name task."
}
```

输出：

```shell
$ gradle -q hello
Hello world!
Greetings from the hello task.
```

### 自定义属性

```groovy
task myTask {
    ext.myProperty = "myValue"
}

task printTaskProperties << {
    println myTask.myProperty

```

输出：

```shell
$ gradle -q printTaskProperties
myValue
```

### 调用 Ant 任务

使用 AntBuilder 来执行 ant.loadfile 任务：

```groovy
task loadfile << {
    def files = file('../antLoadfileResources').listFiles().sort()
    files.each { File file ->
        if (file.isFile()) {
            ant.loadfile(srcFile: file, property: file.name)
            println " *** $file.name ***"
            println "${ant.properties[file.name]}"
        }
    }
}
```

输出：

```shell
$ gradle -q loadfile
*** agile.manifesto.txt ***
Individuals and interactions over processes and tools
Working software over comprehensive documentation
Customer collaboration  over contract negotiation
Responding to change over following a plan
 *** gradle.manifesto.txt ***
```

### 方法

```groovy
task checksum << {
    fileList('../antLoadfileResources').each {File file ->
        ant.checksum(file: file, property: "cs_$file.name")
        println "$file.name Checksum: ${ant.properties["cs_$file.name"]}"
    }
}

task loadfile << {
    fileList('../antLoadfileResources').each {File file ->
        ant.loadfile(srcFile: file, property: file.name)
        println "I'm fond of $file.name"
    }
}

File[] fileList(String dir) {
    file(dir).listFiles({file -> file.isFile() } as FileFilter).sort()
}
```

输出：

```shell
$ gradle -q loadfile
I'm fond of agile.manifesto.txt
I'm fond of gradle.manifesto.txt
```

### 默认任务

Gradle 允许在脚本中定义一个或多个默认任务，构建时自动执行：

```groovy
defaultTasks 'clean', 'run'

task clean << {
    println 'Default Cleaning!'
}

task run << {
    println 'Default Running!'
}

task other << {
    println "I'm not a default task!"
}
```

输出：

```shell
$ gradle -q
Default Cleaning!
Default Running!
```

### DAG 配置

distribution 任务和 release 任务将根据变量的版本产生不同的值：

```groovy
task distribution << {
    println "We build the zip with version=$version"
}

task release(dependsOn: 'distribution') << {
    println 'We release now'
}

gradle.taskGraph.whenReady {taskGraph ->
    if (taskGraph.hasTask(release)) {
        version = '1.0'
    } else {
        version = '1.0-SNAPSHOT'
    }
}
```

输出：

```shell
$ gradle -q distribution
We build the zip with version=1.0-SNAPSHOT
Output of gradle -q release
$ gradle -q release
We build the zip with version=1.0
We release now
```
