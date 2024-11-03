---
title: Flutter 数据/状态共享
toc: true
categories: [Framework,Flutter]
tags: Flutter
date: 2022-07-22 08:28
updated: 2022-07-22 09:43
references:
	- title: 《Flutter实战 第二版》
	  url: https://book.flutterchina.club/chapter7/provider.html
	- title: Flutter Provider
	  url: https://github.com/rrousselGit/provider/blob/master/resources/translations/zh-CN/README.md
---

共享原则：

- 如果状态是组件私有的，则应该由组件自己管理；
- 如果状态要跨组件共享，则该状态应该由各个组件共同的父元素来管理。

<!-- more -->

## InheritedWidget

`InheritedWidget` 是 Flutter 中非常重要的一个功能型组件，它提供了一种在 widget 树中从上到下共享数据的方式。

> `InheritedWidget` 的在 widget 树中数据传递方向是从上到下的，这和通知 `Notification`（将在下一章中介绍）的传递方向正好相反。

下面是“计数器”示例应用程序的 `InheritedWidget` 版本：

<p class="codepen" data-height="566.1328125" data-theme-id="dark" data-default-tab="js,result" data-slug-hash="vYRZYbM" data-preview="true" data-user="sukievaa" style="height: 566.1328125px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/sukievaa/pen/vYRZYbM">
  InheritedWidget</a> by SukiEva (<a href="https://codepen.io/sukievaa">@sukievaa</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

这样，子组件可以调用父组件里的 data 值进行操作。

### didChangeDependencies()

一般来说，子 widget 很少会重写此方法，因为在依赖改变后 Flutter 框架也都会调用 `build()` 方法重新构建组件树。

但是，如果需要在依赖改变后执行一些昂贵的操作，比如网络请求，这时最好的方式就是在此方法中执行，这样可以避免每次 `build()` 都执行这些昂贵操作。

如果只想在 `__TestWidgetState` 中引用 `ShareDataWidget` 数据，但却不希望在 `ShareDataWidget` 发生变化时调用 `__TestWidgetState` 的 `didChangeDependencies()` 方法，将 `ShareDataWidget.of()` 修改一下：

```dart
static ShareDataWidget of(BuildContext context) {
  //return context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
  return context.getElementForInheritedWidgetOfExactType<ShareDataWidget>().widget;
}
```

`dependOnInheritedWidgetOfExactType()` 比 `getElementForInheritedWidgetOfExactType()` 多调了 `dependOnInheritedElement` 方法，`dependOnInheritedElement` 源码如下：

```dart
  @override
  InheritedWidget dependOnInheritedElement(InheritedElement ancestor, { Object aspect }) {
    assert(ancestor != null);
    _dependencies ??= HashSet<InheritedElement>();
    _dependencies.add(ancestor);
    ancestor.updateDependencies(this, aspect);
    return ancestor.widget;
  }
```

**调用 `dependOnInheritedWidgetOfExactType()` 和 `getElementForInheritedWidgetOfExactType()` 的区别：**
**前者会注册依赖关系，而后者不会**

## Provider

对 [InheritedWidget](https://api.flutter-io.cn/flutter/widgets/InheritedWidget-class.html) 组件的上层封装，使其更易用，更易复用。

使用 `provider` 而非手动书写 [InheritedWidget](https://api.flutter-io.cn/flutter/widgets/InheritedWidget-class.html)，有以下的优势:

- 简化的资源分配与处置
- 懒加载
- 创建新类时减少大量的模板代码
- 支持 DevTools
- 更通用的调用 [InheritedWidget](https://api.flutter-io.cn/flutter/widgets/InheritedWidget-class.html) 的方式（参考 [Provider.of](https://pub.flutter-io.cn/documentation/provider/latest/provider/Provider/of.html)/[Consumer](https://pub.flutter-io.cn/documentation/provider/latest/provider/Consumer-class.html)/[Selector](https://pub.flutter-io.cn/documentation/provider/latest/provider/Selector-class.html)）
- 提升类的可扩展性，整体的监听架构时间复杂度以指数级增长（如 [ChangeNotifier](https://api.flutter-io.cn/flutter/foundation/ChangeNotifier-class.html)， 其复杂度为 O(N)）

更多 `provider` 相关内容，请参考 [文档](https://pub.flutter-io.cn/documentation/provider/latest/provider/provider-library.html)。

{% link Flutter Provider 使用::https://www.jianshu.com/p/db5a42074cdd %}
