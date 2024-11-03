---
title: Android ViewBinding
date: 2021-09-27 22:29
tags:
  - Component
categories: Android
toc: true
updated: 2021-09-27 22:29
references:
  - title: 视图绑定  | Android 开发者  | Android Developers
    url: https://developer.android.com/topic/libraries/view-binding?hl=zh-cn
  - title: Android | ViewBinding 与 Kotlin 委托双剑合璧 - 掘金 (juejin.cn)
    url: https://juejin.cn/post/6960914424865488932
---
通过视图绑定功能，可以更轻松地编写可与视图交互的代码。在模块中启用视图绑定之后，系统会为该模块中的每个 XML 布局文件生成一个绑定类。绑定类的实例包含对在相应布局中具有 ID 的所有视图的直接引用。

在大多数情况下，视图绑定会替代 `findViewById`。

<!-- more -->

## 实现原理

在 Android 开发更新迭代中，Kotlin 逐渐取代 Java 成为主流开发语言，原来一些旧的使用方法也随之被取代。`Android-Kotlin-Extensions` 因为安全问题被官方废弃，随之就有了 `view binding`（视图绑定）来继承对 `findViewById` 的替换。

**用于更加轻量地实现视图绑定（视图与变量的绑定），可以理解为轻量版本的 DataBinding**。

Android Gradle 插件会为每个 XML 布局文件创建一个绑定类，绑定类中包含布局文件中每个定义了 `android:id` 属性的 View 引用。假设布局文件为 `fragment_test.xml`，则生成绑定类 `FragmentTestBinding.java`；

## 使用方法

### 开启视图绑定

在模块的 `build.gradle/.kts` 中加入以下内容开启：

```kotlin
android {
   ...
   buildFeatures {
    	viewBinding = true
   }
}
```

如果希望生成绑定类时忽略某个布局文件，在相应的 XML 里加入：

```xml
<LinearLayout
	...
	tools:viewBindingIgnore="true" >
	...
</LinearLayout> 
```

### Activity 中使用

在 Activity 的 [`onCreate()`](https://developer.android.com/reference/kotlin/android/app/Activity?hl=zh-cn#oncreate) 方法中执行以下步骤：

1. 调用生成的绑定类中包含的静态 `inflate()` 方法。此操作会创建该绑定类的实例以供 Activity 使用。
2. 通过调用 `getRoot()` 方法或使用 [Kotlin 属性语法](https://kotlinlang.org/docs/reference/properties.html#declaring-properties) 获取对根视图的引用。
3. 将根视图传递到 [`setContentView()`](https://developer.android.com/reference/kotlin/android/app/Activity?hl=zh-cn#setcontentview_1)，使其成为屏幕上的活动视图。

```kotlin
    private lateinit var binding: ResultProfileBinding

    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)
        binding = ResultProfileBinding.inflate(layoutInflater)
        val view = binding.root
        setContentView(view)
    }

	// 后续通过以下方法引用view
	binding.name.text = viewModel.name
    binding.button.setOnClickListener { viewModel.userClicked() }
```

### Fragment 中使用

在 Fragment 的 [`onCreateView()`](https://developer.android.com/reference/kotlin/androidx/fragment/app/Fragment?hl=zh-cn#oncreateview) 方法中执行以下步骤：

1. 调用生成的绑定类中包含的静态 `inflate()` 方法。此操作会创建该绑定类的实例以供 Fragment 使用。
2. 通过调用 `getRoot()` 方法或使用 [Kotlin 属性语法](https://kotlinlang.org/docs/reference/properties.html#declaring-properties) 获取对根视图的引用。
3. 从 `onCreateView()` 方法返回根视图，使其成为屏幕上的活动视图。

**注意**：`inflate()` 方法会要求您传入布局膨胀器。如果布局已膨胀，可以调用绑定类的静态 `bind()` 方法。如需了解详情，请查看 [视图绑定 GitHub 示例中的例子](https://github.com/android/architecture-components-samples/blob/master/ViewBindingSample/app/src/main/java/com/android/example/viewbindingsample/BindFragment.kt#L36-L41)。

```kotlin
    private var _binding: ResultProfileBinding? = null
    // This property is only valid between onCreateView and
    // onDestroyView.
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = ResultProfileBinding.inflate(inflater, container, false)
        val view = binding.root
        return view
    }
	
	// Fragment 的存在时间比其视图长。请务必在 Fragment 的 onDestroyView() 方法中清除对绑定类实例的所有引用。
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
	// 后续使用
	binding.name.text = viewModel.name
    binding.button.setOnClickListener { viewModel.userClicked() }
```

> **为什么 Fragment#onDestroyView() 里需要置空绑定类对象，而 Activity 里不需要？**
>
> 答：Activity 实例和 Activity 视图的生命周期是同步的，而 Fragment 实例和 Fragment 视图的生命周期并不是完全同步的，因此需要在 Fragment 视图销毁时，手动回收绑定类对象，否则造成内存泄露。
>
> 例如：detach Fragment，或者 remove Fragment 并且事务进入返回栈，此时 Fragment 视图销毁但 Fragment 实例存在。
>
> 总之，在视图销毁但是控制类对象实例还存活的时机，你就需要手动回收绑定类对象，否则造成内存泄露。

## 与其他方案对比

<table>
	<thead>
		<tr>
			<th align="left">角度</th>
			<th align="left">findViewById</th>
			<th align="left">ButterKnife</th>
			<th align="left">Kotlin Synthetics</th>
			<th align="left">DataBinding</th>
			<th align="left">ViewBinding</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td align="left">简洁性</td>
			<td align="left">✖</td>
			<td align="left">✖</td>
			<td align="left">✔</td>
			<td align="left">✔</td>
			<td align="left">✔</td>
		</tr>
		<tr>
			<td align="left">编译期检查</td>
			<td align="left">✖</td>
			<td align="left">✖</td>
			<td align="left">✖</td>
			<td align="left">✔</td>
			<td align="left">✔</td>
		</tr>
		<tr>
			<td align="left">编译速度</td>
			<td align="left">✔</td>
			<td align="left">✖</td>
			<td align="left">✔</td>
			<td align="left">✖</td>
			<td align="left">✔</td>
		</tr>
		<tr>
			<td align="left">支持 Kotlin &amp; Java</td>
			<td align="left">✔</td>
			<td align="left">✔</td>
			<td align="left">✖</td>
			<td align="left">✔</td>
			<td align="left">✔</td>
		</tr>
		<tr>
			<td align="left">收敛模板代码</td>
			<td align="left">✖</td>
			<td align="left">✖</td>
			<td align="left">✔</td>
			<td align="left">✖</td>
			<td align="left">✖</td>
		</tr>
	</tbody>
</table>

- **简洁性：** findViewById 和 ButterKnife 需要在代码中声明很多变量，其他几种方案代码简洁读较好；
- **编译检查：** 编译期间主要有两个方面的检查：类型检查 + 只能访问当前布局中的 id。findViewById、ButterKnife 和 Kotlin Synthetics 在这方面表现较差；
- **编译速度：** findViewById 的编译速度是最快的，而 ButterKnife 和 DataBinding 中存在注解处理，编译速度略逊色于 Kotlin Synthetics 和 ViewBinding；
- **支持 Kotlin & Java：** Kotlin Synthetics 只支持 Kotlin 语言；
- **收敛模板代码：** 基本上每种方案都带有一定量的模板代码，只有 Kotlin Synthetics 的模板代码是较少的。
