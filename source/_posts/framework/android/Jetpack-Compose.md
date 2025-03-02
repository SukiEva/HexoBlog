---
title: Jetpack Compose
date: 2021-09-29 21:30
tags:
  - Jetpack
categories: [Framework,Android]
toc: true
updated: 2021-09-29 21:30
---

使用 JetpackCompose 的一些笔记。

<!-- more -->

**关键术语** - **组合**：Jetpack Compose 在执行可组合项时构建的界面描述。
**初始组合**：通过首次运行可组合项创建组合。
**重组**：在数据发生变化时重新运行可组合项以更新组合。

`  'Surface' A surface container using the 'background' color from the theme `

`'Divider' is a provided composable function that creates a horizontal divider.`

## 可组合项中的状态

可组合函数可以使用 `remember` 可组合项记住单个对象。

系统会在初始组合期间将由 `remember` 计算的值存储在组合中，并在重组期间返回存储的值。

- `remember` 既可用于存储可变对象，又可用于存储不可变对象。
- `mutableStateOf` 会创建可观察的 `MutableState` 会创建可观察的 `MutableState`，后者是与 Compose 运行时集成的可观察类型。

在可组合项中声明 `MutableState` 对象的方法有三种（等价）：

- `val mutableState = remember { mutableStateOf(default) }`
- `var value by remember { mutableStateOf(default) }`
- `val (value, setValue) = remember { mutableStateOf(default) }`

```kotlin
@Composable
fun Counter() {
    val count = remember { mutableStateOf(0) }
    Button(onClick = { count.value++ }) {
        Text("I've been clicked ${count.value} times")
    }
}
```

## Color

<center><img src="https://developer.android.com/codelabs/jetpack-compose-theming/img/bb8ab0b2d8f9bca8.png?hl=zh-cn" alt="Material Design Color"  /></center>

- `primary`：main brand color
- `secondary`：provide accents

## Typography

<center><img src="https://developer.android.com/codelabs/jetpack-compose-theming/img/767fd40cb6938dc4.png?hl=zh-cn" alt="Typographies" style="zoom: 50%;" /></center>
