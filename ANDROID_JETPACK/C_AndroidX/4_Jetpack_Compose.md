# Jetpack 组件
> 原文链接：[Jetpack Compose | Android Developers](https://developer.android.google.cn/jetpack/compose)

**用于构建 UI 的声明性工具包**

Jetpack 组件（Compose）是一个非捆绑工具包，旨在简化 UI 开发。它结合了响应式编程模型和 Kotlin 编程语言的简洁性及易用性。

> **注意:** Jetpack 组件目前处于早期探索的 pre-alpha 阶段。它的 API 层面尚未最终确定，不应用于生产。

[探索 JETPACK COMPOSE](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/ui/README.md)

## 核心理念

**简洁易用的Kotlin**

基于 Kotlin 带来的好处——简洁、安全、并与Java完全互操作。旨在大幅减少必须编写的样板代码量，以便您可以专注于应用程序代码，并帮您避免各种类型的错误。

**声明性**

完全声明用于定义 UI 组件，包括绘制和创建自定义布局。只需将 UI 描述为一组可组合函数，框架就可以处理 UI 优化并自动更新视图层次结构。

**兼容性**

与现有视图兼容，因此您可以混合搭配，并按照自己的意愿采用直接访问所有 Android 和 Jetpack 的 API。

**创造漂亮的应用**

从一开始就采用开箱即用的材料化设计（Material Design）和动画设计，因此可以轻松创建充满动感的漂亮应用程序。

**加速开发**

通过编写更少的代码并使用 Apply Changes 和实时预览等工具来加速开发。

## 快速浏览

Jetpack 组件正在 [Android开源项目](https://android.googlesource.com/platform/frameworks/support/) 中开发。它有两个主要组成部分：
- 组件 UI 库，其中包含核心的 UI 工具包，包括布局、输入、文本、动画、样式、小部件和图形。
- 组件编译器，一个自定义的 Kotlin 编译器插件，它采用可组合的函数，并自动更新 UI 层次结构。

组件（Compose）应用程序由可组合函数构成，可将应用程序数据转换为 UI 层次结构。只需一个函数即可创建新的 UI 组件。

要创建可组合函数，只需将 `@Composable` 注解添加到函数名上即可。在底层，组件使用自定义的 Kotlin 编译器插件，因此当底层数据发生更改时，可以重新调用可组合函数以生成更新的 UI 层次结构。下面的简单示例将一个字符串打印到屏幕上。

```kotlin
import androidx.compose.*
import androidx.ui.core.*

@Composable
fun Greeting(name: String) {
   Text ("Hello $name!")
}
```

Jetpack 组件 UI 库的 API 位于 AOSP [frameworks/support/ui](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/ui/) 目录中。组件编译器和运行时代码可以在 [frameworks/support/compose](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/compose/) 中找到。

## 组件 UI 库

Jetpack 组件 UI 库包含以下模块：

**android-text/**

Android 特定于文本堆栈的实现

**android-view/**

现有 Android 视图的包装器和适配器

**animation/**

动画组件

**animation-core/**

动画系统的内部声明

**core/**

系统中使用的基类包括基本元素、图形和绘画

**framework/**

基本组件由系统作为构建块公开。这包括绘图、布局、文本等。

**layout/**

基本布局组件

**material/**

根据材料化设计规范构建的 UI 组件集

**platform/**

内部实现，允许 Android 实现与主机端测试分离

**test/**

测试框架

**text/**

文本引擎
