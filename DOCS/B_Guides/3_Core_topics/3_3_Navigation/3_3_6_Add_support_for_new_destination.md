# 支持新的目的地类型
> 原文链接：[Add support for new destination types  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation/navigation-add-new)

[NavController](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn) 依赖一个或多个 [Navigator](https://developer.android.google.cn/reference/androidx/navigation/Navigator.html?hl=zh-cn) 对象来实行导航操作。

在默认情况下，所有的 [NavController](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn) 都支持使用 [`ActivityNavigator`](https://developer.android.google.cn/reference/androidx/navigation/ActivityNavigator.html?hl=zh-cn) 类及其内部类 [ActivityNavigator.Destination](https://developer.android.google.cn/reference/androidx/navigation/ActivityNavigator.Destination.html?hl=zh-cn) 导航到另一个 activity 的方式，来离开当前的导航图。

欲导航到其他类型的目的地，您必须为 [NavController](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn) 添加一个或多个新的 [Navigator](https://developer.android.google.cn/reference/androidx/navigation/Navigator.html?hl=zh-cn)。例如，当您把 fragment 作为目的地时，[`NavHostFragment`](https://developer.android.google.cn/reference/androidx/navigation/fragment/NavHostFragment.html?hl=zh-cn) 会自动为其 [NavController](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn) 添加 [`FragmentController`](https://developer.android.google.cn/reference/androidx/navigation/fragment/FragmentNavigator.html?hl=zh-cn) 类。

欲为 [NavController](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn) 添加新的 [Navigator](https://developer.android.google.cn/reference/androidx/navigation/Navigator.html?hl=zh-cn) 对象，您必须使用各自 `NavController` 类的 [`getNavigatorProvider`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#getNavigatorProvider()) 方法，接着再调用其 [`addNavigator()`](https://developer.android.google.cn/reference/androidx/navigation/NavigatorProvider.html?hl=zh-cn) 方法。下例展示如何将一个虚构的 `CustomNavigator` 添加到一个 [NavController](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn)：

```java
CustomNavigator customNavigator = new CustomNavigator();
navController.getNavigatorProvider().addNavigator(customNavigator);
```

```kotlin
val customNavigator = CustomNavigator()
navController.navigatorProvider += customNavigator
```

大多数 [Navigator](https://developer.android.google.cn/reference/androidx/navigation/Navigator.html?hl=zh-cn)  都有一个目的地的内部类，该内部类可以被用来指定和您目的地所独有的特定属性。欲了解更多有关目的地内部类的信息，请参阅相应 [Navigator](https://developer.android.google.cn/reference/androidx/navigation/Navigator.html?hl=zh-cn) 类的参考文档。

