---
title: 主流开源协议
toc: true
date: 2021-11-12 18:49
tags:
  - License
categories: Mixed
updated: 2021-11-12 18:49
references:
  - title: 主流开源协议之间有何异同？ - 知乎 (zhihu.com)
    url: https://www.zhihu.com/question/19568896
  - title: 如何选择开源许可证？ - 阮一峰的网络日志 (ruanyifeng.com)
    url: https://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html
---

常见的开源许可证主要有 Apache、MIT、BSD、GPL、LGPL、MPL、SSPL 等，可以大致分为两大类：

- 宽松自由软件许可协议 *Permissive free software licence*
- 著佐权许可证 *copyleft license*

其中，Apache、MIT、BSD 都是宽松许可证，GPL 是典型的强著佐权（copyleft ）许可证，LGPL、MPL 是弱著佐权（copyleft ）许可证。

<!-- more -->

## 区别

- Permissive free software licence ：

	一种对软件的使用、修改、传播等方式采用最低限制的自由软件许可协议条款类型。这种类型的软件许可协议将不保证原作品的派生作品会继续保持与原作品完全相同的相关限制条件，从而为原作品的自由使用、修改和传播等提供更大的空间。

- Copyleft License ：

	在有限空间内的自由使用、修改和传播，且不得违背原作品的限制条款。如果一款软件使用 Copyleft 类型许可协议规定软件不得用于商业目的，且不得闭源，那么后续的衍生子软件也必须得遵循该条款。

两者最大的差别在于：在软件被修改并再发行时， Copyleft License 仍然强制要求公开源代码（衍生软件需要开源），而 Permissive free software licence 不要求公开源代码（衍生软件可以变为专有软件）。

其中，Apache、MIT、BSD 都是宽松许可证，GPL 是典型的强著佐权（copyleft ）许可证，LGPL、MPL 是弱著佐权（copyleft ）许可证。

## 常见开源许可证

<img src="https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/free_software_licenses.png" alt=" 常见许可证 " style="zoom:50%;" />

### MIT

MIT 许可证是史上最为简洁和慷慨（permissive）的开源协议之一。应用案例有：JQuery、Rails 等。

### BSD

*Berkeley Software Distribution license*

BSD 许可证与 MIT 差不多。

事实上，BSD 许可证被认为是 copycenter（中间著作权），介乎标准的 copyright 与 GPL 的 copyleft 之间。

可以说，GPL 强迫后续版本必须一样是自由软件，BSD 的后续版本可以选择要继续是 BSD 或其他自由软件条款或闭源软件等等。

### Apache

Apache 2 的效力与 MIT 近似，区别主要在于前者额外提供了一份简易的专利许可授权，明确禁止商标使用权以及要求明确指明所有修改过的文件（state changes）。

Apache 许可证要求被授权者保留著作权和放弃权利的声明，但它不是一个反著作权的许可证。此许可证最新版本为 “版本 2”，于 2004 年 1 月发布。

Apache 许可证是宽松的，因为它不会强制派生和修改作品使用相同的许可证进行发布。

### GPL

GPL 的出发点是代码的开源/免费使用和引用/修改/衍生代码的开源/免费使用，但不允许修改后和衍生的代码做为闭源的商业软件发布和销售。

由于 GPL 严格要求使用了 GPL 类库的软件产品必须使用 GPL 协议，对于使用 GPL 协议的开源代码，商业软件或者对代码有保密要求的部门就不适合集成/采用此作为类库和二次开发的基础。

![简明对比](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/7149baf878e23293b9bd57df076b6e41_r.jpg)

### LGPL

更宽松的 GPL 协议。

与 GPL 协议的区别为，后者如果只是对 LGPL 软件的程序库的程序进行调用而不是包含其源代码时，相关的源程序无需开源。

调用和包含的区别类似在互联网网网页上对他人网页内容的引用：如果把他人的内容全部或部分复制到自己的网页上，就类似包含，如果只是贴一个他人网页的网址链接而不引用内容，就类似调用。有了这个协议，很多大公司就可以把很多自己后续开发内容的源程序隐藏起来。
