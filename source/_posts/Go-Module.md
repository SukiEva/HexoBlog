---
title: Go Module
toc: true
date: 2021-11-17 08:54
tags:
  - Go
categories: [Programming Language,Go]
updated: 2021-11-17 08:54
references:
  - title: 06｜构建模式：Go是怎么解决包依赖管理问题的？ (geekbang.org)
    url: https://time.geekbang.org/column/article/429941
---

从 Go 1.11 版本开始，除了 GOPATH 构建模式外，Go 又增加了一种 Go Module 构建模式。

在 Go Module 模式下，通常一个代码仓库对应一个 Go Module。

一个 Go Module 的顶层目录下会放置一个 go.mod 文件，每个 go.mod 文件会定义唯一一个 module，也就是说 Go Module 与 go.mod 是一一对应的。

go.mod 文件所在的顶层目录也被称为 module 的根目录，module 根目录以及它子目录下的所有 Go 包均归属于这个 Go Module，这个 module 也被称为 main module。

<!-- more -->

## 前言

Go 的项目依赖管理经历了从 `Go Path => Go Vendor => Go Moudle ` 的演变。

### Go Path

`Go Path` 主要目录结构如下：

```shell
.
|__bin	# 项目编译后的二进制文件
|__pkg	# 项目编译中间产物，加速编译
|__src	# 项目源码
```

然而，当遇到 A 和 B 依赖于某一个包的不同版本时，无法实现 package 的多版本控制。

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202206172150948.png" style="zoom:67%;" />

### Go Vendor

在 `Go Path` 的目录结构中，新增了 `vendor` 目录，如果存在该目录，优先使用其下的依赖，否则从 `Go Path` 中寻找。

但是，`Go Vendor` 不能很清晰地标识版本的概念：

- 无法控制依赖的版本
- 更新项目可能出现依赖冲突，导致编译出错

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202206172154042.png" style="zoom:67%;" />

## Go Module 构建模式

### 语义导入版本机制

在 Go Module 构建模式下，一个符合 Go Module 要求的版本号，由前缀 v 和一个满足语义版本规范的版本号组成

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/468323b3294cce2ea7f4c1da3699c5a2.png" alt=" 语义版本号规范 " style="zoom: 50%;" />

按照语义版本规范，

- **主版本号不同**的两个版本是相互**不兼容**的；
- 在主版本号相同的情况下，次版本号大都是**向后兼容**次版本号小的版本；
- 补丁版本号也不影响兼容性。

Go Module 规定：如果同一个包的新旧版本是兼容的，那么它们的包导入路径应该是相同的。

```go
// 通过在包导入路径中引入主版本号的方式，来区别同一个包的不兼容版本
// 假如这是 v1.x.x 版本
import "github.com/sirupsen/logrus" 
// 如果要导入 v2.x.x 版本
import "github.com/sirupsen/logrus/v2"
// 甚至可以同时依赖一个包的两个不兼容版本
import (
    "github.com/sirupsen/logrus"
    logv2 "github.com/sirupsen/logrus/v2"
)
```

> Go Module 将这样的版本 (v0) 与主版本号 v1 做同等对待，也就是采用不带主版本号的包导入路径

### 最小版本选择原则

依赖关系一旦复杂起来，比如像下图中展示的这样，Go 又是如何确定使用依赖包 C 的哪个版本的呢？

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/49eb7aa0458d8ec6131d9e5661155f1b.jpeg" alt=" 复杂情况 " style="zoom: 33%;" />

当前存在的主流编程语言，以及 Go Module 出现之前的很多 Go 包依赖管理工具都会选择依赖项的“最新最大 (Latest Greatest) 版本”，对应到图中的例子，这个版本就是 `v1.7.0`。

不过，Go 会在该项目依赖项的所有版本中，选出**符合项目整体要求**的“最小版本”

这个例子中，`C v1.3.0` 是符合项目整体要求的版本集合中的版本最小的那个，于是 Go 命令选择了 `C v1.3.0`，而不是最新最大的 `C v1.7.0`。

## Go Module 版本管理

### （1）Go Module 创建

创建一个 Go Module，通常有如下几个步骤：

1. 通过 `go mod init` 创建 go.mod 文件，将当前项目变为一个 Go Module；
2. 通过 `go mod tidy` 命令自动更新当前 module 的依赖信息；
3. 执行 `go build`，执行新 module 的构建。

### （2）为当前 Module 添加一个依赖

```shell
# 手动添加
$ go get github.com/google/uuid

# 自动添加
$ go mod tidy
```

### （3）升级 / 降级依赖的版本

```shell
# 选定指定版本即可
$go get github.com/sirupsen/logrus@v1.7.0
```

### （4）移除一个依赖

仅从源码中删除对依赖项的导入语句还不够，还得用 `go mod tidy` 命令，将这个依赖项彻底从 Go Module 构建上下文中清除掉。

`go mod tidy` 会自动分析源码依赖，而且将不再使用的依赖从 go.mod 和 go.sum 中移除。

### （5）vendor

vendor 机制可以对 vendor 目录下缓存的依赖包进行自动管理。

在 Go 1.14 及以后版本中，如果 Go 项目的顶层目录下存在 vendor 目录，那么 go build 默认也会优先基于 vendor 构建，除非 go build 传入 `-mod=mod` 的参数。

```shell
$go mod vendor
$tree -LF 2 vendor
vendor
├── github.com/
│   ├── google/
│   ├── magefile/
│   └── sirupsen/
├── golang.org/
│   └── x/
└── modules.txt
```

### 空导入

像下面代码这样的包导入方式被称为“空导入”：`import _ "foo"`

空导入也是导入，意味着我们将依赖 foo 这个路径下的包。

由于是空导入，我们并没有显式使用这个包中的任何语法元素。

通常实践中空导入意味着期望依赖包的 init 函数得到执行，这个 init 函数中有我们需要的逻辑。
