# 开始动手
> 原文链接：[Get started  |  Android Developers](https://developer.android.google.cn/topic/libraries/data-binding/start)

本小节您将学习如何准备好包括设置 Android Studio 在内的开发环境，用于使用数据绑定（Data Binding）库。

数据绑定库既提供了灵活性，又满足了广泛的实用性——它是一个支持库，因此您可以在运行 Android 4.0 （API 版本 14）及更新系统的设备上使用它。

我们推荐您在项目中使用最新版本的 Android Gradle 插件，然而，只有 1.5.0 及更高版本的插件才支持数据绑定。欲了解更多信息，请参阅[升级 Android Gradle 插件](https://developer.android.google.cn/studio/releases/gradle-plugin#updating-plugin)。

## 构建环境

欲开始使用数据绑定，请在 Android SDK 管理器中从**支持库代码仓库**下载该库。欲了解更多信息，请参阅[更新 IDE 和 SDK 工具](https://developer.android.google.cn/studio/intro/update)*（请选择简体中文）*。

欲配置您的应用来使用数据绑定，向您的应用模块的 `build.gradle` 中加入 `dataBinding` 元素，如下所示：

```gradle
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

> **注意**：即使应用模块并不直接使用数据绑定，您也必须为那些使用了依赖于数据绑定库的模块配置数据绑定。

## Android Studio 对于数据绑定的支持

Android Studio 支持数据绑定代码的多种编辑功能。如下所示：

* 语法高亮
* 表达式语法错误标记
* XML 代码补全
* 参考，包括[导航](https://www.jetbrains.com/help/idea/2017.1/navigation-in-source-code.html)（例如跳转到定义）和[快速文档](https://www.jetbrains.com/help/idea/2017.1/viewing-inline-documentation.html)。

> **警告**：数组和[泛型](https://docs.oracle.com/javase/tutorial/java/generics/types.html)，例如 [`Observable`](https://developer.android.google.cn/reference/android/databinding/Observable`) 类，可能不正确地显示错误信息。

只要您提供了数据绑定表达式的缺省值的话，**布局编辑器（Layout Editor）**中的**预览（Preview）**窗格就能展示该缺省值。例如，**预览（Preview）**窗格在 `TextView` 部件上展示了值 `你瞅啥`，如下例所示：

```xml
<TextView android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.firstName, default=你瞅啥}"/>
```

如果您只在项目的设计阶段才需要展示缺省值，那么请使用 `tools` 属性，而非表达式的缺省值，如 [Tools 属性参考](https://developer.android.google.cn/studio/write/tool-attributes)所述。

## 新的数据绑定编译器

`3.1.0-alpha06` 版本的 Android Gradle 插件包含了一个新的数据绑定编译器，用于生成绑定类。新的编译器增量地创建 binding 类，这在大多数情况下都能加速构建过程。欲了解更多有关 binding 类的信息，请参阅[自动生成的绑定类](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_5_Generated_binding_classes.md)。

之前版本的数据绑定编译器的编译步骤和您自己编写的代码的编译步骤相同。如果您自己编写的代码编译失败了，您可能会得到多个报告“绑定类未找到”的错误。而新的编译器会在编译您的代码之前就生成绑定类，从而防止了这些错误。

欲启用新的数据绑定编译器，请将如下配置代码加入您的 `gradle.properties` 文件：

```gradle
android.databinding.enableV2=true
```

您还能通过添加如下参数来在您的 Gradle 命令中启用新的编译器：

```gradle
-Pandroid.databinding.enableV2=true
```

> **注意**：Android Studio 3.1 所包含的新的数据绑定编译器并不向后兼容。欲利用增量编译的特性，您需要启用该功能来生成您的所有绑定类。然而，3.2 版本的 Android Gradle 插件所包含的新的编译器能够向后兼容之前版本所生成的绑定类，该编译器在缺省条件下是启用的。

当您启用新的数据绑定编译器时，会有如下的变化：

* Android Gradle 插件在编译您自己编写的代码之前就为您的布局生成绑定类。
* 如果一个布局被多个目标资源配置（target resource configuration）所包含，那么数据绑定库会将 `android.view.View` 作为所有资源 id 相同但视图类型不同的视图的默认视图类型。
* 类库模块的绑定类会被编译并打包到相应的 AAR 文件中。依赖于这些类库模块的应用模块无须再次生成这些绑定类。欲了解更多有关 AAR 文件的信息，请参阅[创建 Android 库](https://developer.android.google.cn/studio/projects/android-library)*（请选择简体中文）*。
* 一个模块的绑定适配器（binding adapter）不能再改变该模块的依赖的适配器行为。绑定适配器只会影响其模块及其消费者的代码。





