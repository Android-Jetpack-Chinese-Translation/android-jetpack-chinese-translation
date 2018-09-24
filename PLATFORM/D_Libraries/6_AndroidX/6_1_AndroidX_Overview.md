# AndroidX 概述
> 原文链接：[AndroidX Overview  |  Android Developers](https://developer.android.google.cn/topic/libraries/support-library/androidx-overview)

我们正在推出一个新的包结构，用来清晰地表明哪些包是和 Android 操作系统捆绑在一起的、哪些又是打包到您应用的 APK 文件中的。在未来，[`android.* `] 包层次结构将被保留为和操作系统一起推出的 Android 包，而其他的包将会被指定为新的 [`androidx.*`] 包层次结构。

现有的包层次结构将被重构来使用新的层级结构。早期历史版本的 artifact ——即早于修订版本27的，以及打包入 `android.support.*` 的——将在 Google Maven 上保持可用。然而，所有新的开发都会在新的 `androidx.*` 打包的 artifact 中，其版本号从 1.0.0 开始。

欲了解类和构建 artifact 从旧到新的完整映射关系，请参阅 [AndroidX 重构](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/D_Libraries/6_AndroidX/6_2_Refactoring.md)。欲了解有关 AndroidX 重构的更多信息，请参阅[该博客](https://android-developers.googleblog.com/2018/05/hello-world-androidx.html)。

## 版本号变化

新的 artifact 将遵循[语义化版本（Semantic Versioning）](https://semver.org/lang/zh-CN/)的规范，并各自独立（而非一起同时）更新。通过这样的重组，您就能分别更新项目中的每个 AndroidX 类库，从而避免了同时更新您项目中大量的支持库模块，譬如把所有的 `26.1.0` 一下升级到 `27.0.0`。

## 新项目

如果您想创建一个依赖于 AndroidX 架构的新项目——而非用 Android Studio 重构一个现有的项目——那么这个新项目需要将目标 API 等级设为 28，并且您需要将如下的代码加入您的 [`gradle.properties` 文件](https://developer.android.google.cn/studio/build/#properties-files)：

```gradle
android.useAndroidX=true
android.enableJetifier=true
```


