# AndroidX 概览

> 原文链接：[AndroidX Overview | Android Developers](https://developer.android.google.cn/jetpack/androidx)

AndroidX 是 Android 团队用于在 [Jetpack](https://developer.android.google.cn/jetpack) 中开发、测试、打包、发版和发布依赖库的开源项目。

AndroidX 是对原有 Android [支持库](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/D_Libraries/1_Support_Library_Overview.md) 的重大改进。与支持库一样，AndroidX 与 Android 操作系统分开提供，并提供跨 Android 版本的向后兼容性。 AndroidX 通过提供相同功能和新的依赖库完全取代了支持库。此外，AndroidX还包括以下功能：
- AndroidX 中的所有软件包都以字符串 `androidx` 开头，位于一致的命名空间。支持库包已映射到相应的 `androidx*` 包中。有关所有旧类和构件到新版的完整映射，请参阅 [迁移到 AndroidX](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/C_AndroidX/2_Migrating_to_AndroidX.md) 页面。
- 与支持库不同，AndroidX 软件包是单独维护和更新的。从版本 1.0.0 开始，`androidx` 软件包使用严格的 [语义版本控制](https://semver.org/)。您可以单独更新项目中的 AndroidX 库。
- 所有新的支持库开发都将在 AndroidX 库中进行。这包括维护原有支持库和引入新的 Jetpack 组件。

## 使用 AndroidX

请参阅 [迁移到 AndroidX](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/C_AndroidX/2_Migrating_to_AndroidX.md) 以了解如何迁移现有项目。

如果要在新项目中使用 AndroidX，则需要将 compile SDK 设置为 Android 9.0（API级别28）或更高版本，并在 [gradle.properties文件](https://developer.android.google.cn/studio/build/#properties-files) 中将以下两个 Android Gradle 插件标志设置为 `true`。
- `android.useAndroidX`：设置为 `true` 时，Android 插件使用相应的 AndroidX 库而不是支持库。如果未指定，则默认情况下该标志为 `false`。
- `android.enableJetifier`：当设置为 `true` 时，Android 插件会自动迁移现有的第三方库，通过重写其二进制文件来使用 AndroidX。如果未指定，则默认情况下该标志为 `false`。

## AndroidX 参考文档

AndroidX 中的所有包和类都可以在 [AndroidX 参考文档](https://developer.android.google.cn/reference/androidx/packages) 找到。

## 其他资源

Jetpack 组件是 AndroidX 库的一部分。在 [Jetpack主页](https://developer.android.google.cn/jetpack) 上了解有关组件的更多信息。

有关从支持库到 AndroidX 的包重构的更多信息，请参阅 [博客文章](https://android-developers.googleblog.com/2018/05/hello-world-androidx.html)。
