# Navigation 概览
> 原文链接：[Navigation | Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation)

导航是应用设计中的至关重要的部分。使用导航，您可以设计出用户在应用中的各个部分之间跳转、进入和回退的交互。

## Navigation 架构组件

Navigation 架构组件能帮您在应用中实现常见的复杂导航，从而更轻松地为用户提供可预测的连贯体验。

Navigation 处理您应用的 *目的地（destination）* 之间的导航，即：您应用中任何一个用户能导航到的地方。尽管目的地常常是代表特定页面的 `Fragment`，Navigation 组件也支持下列其他目的地类型：

* Activity。
* 导航图（graph）和子图（subgraph）：当目的地是图/子图时，用户被导航到该图/子图的起始目的地。
* [自定义的目的地类型](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_3_Navigation/3_3_6_Add_support_for_new_destination.md)

这些目的地是通过 *动作（action）* 连接起来的。一组目的地及其之间的动作构成了一个应用的 *导航图（navigation graph）*。

Navigation 架构组件还提供了如下所示的其他好处：

* 处理 Fragment transaction。
* 默认情况下就能正确处理“向上”和“向后”行为。
* 为动画和转场（transition）提供标准化的资源。
* 把深层链接（deep link）作为一级操作来对待。
* 以最小的额外代价，提供诸如导航抽屉和底部导航栏等导航 UI 样式。
* 使用 Android Studio 的 [Navigation 编辑器](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_3_Navigation/3_3_2_Implement_Navigation_with_the_Navigation_Architecture_Component.md) 来可视化编辑导航图。

> **注意**：欲在 Android Studio 中使用 Navigation 架构组件，您必须使用 Android Studio 3.3 或更高版本。

## 导航的原则

Navigation 架构组件是基于以下的设计原则：

### 固定的起始目的地

应用应当有一个固定的 *起始目的地（start destination）*。它是用户在启动您的应用时看到的页面，也是用户使用返回键返回到桌面/启动器前所看到的最后一个页面。

> **注意**：应用可能有一个一次性的初始设置，或者一系列的登录页面。这些有条件的页面应当不被认作您应用的起始目的地。

### 导航状态应能表示为一个目的地的堆栈

一个 *导航堆栈（navigation stack）* 的栈底应当是起始目的地，而栈顶则是当前的目的地。

改变导航堆栈的操作应当总是工作在栈顶：它要么是把一个新的目的地 push 到堆栈内，要么则是把最顶端的目的地从堆栈内 pop 出去。

### “向上”按钮永远不会退出应用

如果一个用户正处于起始目的地，那么“向上”按钮就不应显示。当您的应用是被一个深层链接（deeplink）启动到另一个应用的任务上时，“向上”按钮应把用户带到当前页面在您应用架构中的上一层，而不是把用户带回到启动它的另一个应用。

### 深层链接和导航到同一个目的地应当生成相同的堆栈

无论用户是如何到达某个目的地的，他都应当能使用”向后“或”向上“按钮来一步步回到最初的目的地。

当使用深层链接时，任何已经存在的导航堆栈都将被移除，并被深层链接的导航堆栈所代替。

## 开始动手

* [Navigation 代码实验室](https://codelabs.developers.google.com/codelabs/android-navigation?hl=zh-cn)
* [用 Navigation 架构组件来实现导航](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_3_Navigation/3_3_2_Implement_Navigation_with_the_Navigation_Architecture_Component.md)

## 其他资源

* [Android Jetpack：使用 Navigation 控制器来管理 UI 导航](https://www.youtube.com/watch?v=8GCXtCjtg40&hl=zh-cn)
