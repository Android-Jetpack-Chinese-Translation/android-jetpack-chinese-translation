# 为您的项目添加 Android 架构组件
> 原文链接：[Adding Components to your Project  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/adding-components)

在开始之前，我们建议您先阅读这篇[应用架构指南](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/B_Get_started/2_Guide_to_app_architecture.md)。该指南提及了一些对任何应用都有意义的原则，并且展示了应该如何把架构组件整合起来。

Android 架构组件可以从 Google 的 Maven 仓库中获取，要使用组件，您必须将仓库添加到项目中。

打开**您的项目**（而非您的应用或模块）的 `build.gradle` 文件，然后添加 `google()` 仓库，如下所示：

```groovy
allprojects {
    repositories {
        google()
        jcenter()
    }
}
```

## 声明依赖关系

打开**您的应用或模块**的 `build.gradle` 文件，然后添加您需要依赖的 artifact。您可以添加所有架构组件的全部依赖，或者只添加其中的一部分。

请参阅发行日志中有关为每个组件声明依赖关系的说明：
- [WorkManager](https://developer.android.google.cn/jetpack/androidx/releases/work)：后台任务管理
- [Navigation](https://developer.android.google.cn/jetpack/androidx/releases/navigation)（包含 SafeArgs）：导航
- [Paging](https://developer.android.google.cn/jetpack/androidx/releases/paging)：分页
- [Lifecycle components](https://developer.android.google.cn/jetpack/androidx/releases/lifecycle)（包含 ViewModel）：生命周期组件
- [Futures](https://developer.android.google.cn/jetpack/androidx/releases/concurrent) （在 androidx.concurrent 包中）：并发
- [Room](https://developer.android.google.cn/jetpack/androidx/releases/room)（此项为译者补充）：数据持久化

欲了解有关 AndroidX 重构的更多细节、以及它会如何影响包和模块的 ID，请参阅 [AndroidX 重构](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/D_Libraries/6_AndroidX/6_2_Refactoring.md)。

## Kotlin

几个 AndroidX 依赖项已支持 Kotlin 扩展模块。 这些模块的名称后面附加了后缀 “-ktx”。 例如：

```groovy
implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
```

变成

```groovy
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
```

欲了解包括 Kotlin 扩展的更多信息，请参阅 [Android KTX](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/4_Android_KTX.md)。

> **注意**：请务必在基于 Kotlin 的应用中使用 **kapt** 而非 **annotationProcessor**。此外，您还应该添加 **kotlin-kapt** 插件。
