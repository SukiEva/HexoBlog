---
title: Go项目测试
toc: true
categories: [Programming Language,Go]
tags:
  - Go
  - Test
date: 2022-06-17 21:59:55
updated: 2022-06-25 16:38
references:
  - title: 字节跳动青训营
    url: https://juejin.cn/user/3386151545092589
---

测试关系着系统的质量，质量则决定线上系统的稳定性。

<!-- more -->

测试自顶向下分为：

- 回归测试：由 QA 手动通过终端回归一些固定的主流程场景
- 集成测试：对系统功能维度做测试验证
- 单元测试：对单独的函数、模块做功能验证

> 从顶向下，覆盖率逐层增加，成本逐层降低
>
> 单元测试的覆盖率一定程度上决定了代码的质量

## 单元测试
