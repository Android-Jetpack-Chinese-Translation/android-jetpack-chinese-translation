# 支持库软件包
> 原文链接：[Support Library Packages  |  Android Developers](https://developer.android.google.cn/topic/libraries/support-library/packages)

Android 支持库含有若干可导入到您的应用的软件包，它们各自支持一定特定范围内的 Android 平台版本和功能特性。

欲使用下列任意一个库，您必须下载库文件到您的 SDK 安装目录，按照[支持库设置](https://developer.android.google.cn/tools/support-library/setup.html#download)*（请选择简体中文）*的指导进行操作即可。在这之后，您还必须将特定的支持库导入到您的应用中，导入每个库的具体方法请参阅相应段落的结尾。

> **注意**：所有支持库软件包所支持的最低 SDK 版本至少是 API 版本 14，有些软件包要求的更高，下面会在注意事项中一一提及。

## v4 支持库

下列各库相对于其他库而言包含了最多的 API，提供了对应用组件、UI 特性、无障碍、数据处理、网络连接和编程实用程序的支持。

欲获取 v4 支持库提供的类和方法的完整详细信息，请参阅 API 文档中的 [`android.support.v4`](https://developer.android.google.cn/reference/android/support/v4/app/package-summary.html) 软件包。

> **注意**：在支持库修订版本 24.2.0 之前，曾只有一个单独的 v4 支持库，该库后来被分割成若干模块来提升效率。如果您在 Gradle 脚本中列出了 **support-v4**，您生成的 APK 安装包将包含所有 v4 的模块以向后兼容。然而，我们仍然推荐您只列出您应用所需的特定模块，以便减小 APK 安装包的大小。

### v4 兼容库

为若干 Android 框架 API 提供向后兼容的包装类，例如 `Context.obtainDrawable()` 和 `View.performAccessibilityAction()`。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-compat:27.1.1
```

### v4 核心 UI 库

实现了许多和 UI 相关的组件，例如 [`ViewPager`](https://developer.android.google.cn/reference/android/support/v4/view/ViewPager.html)、[`NestedScrollView`](https://developer.android.google.cn/reference/android/support/v4/widget/NestedScrollView.html) 和 [`ExploreByTouchHelper`](https://developer.android.google.cn/reference/android/support/v4/widget/ExploreByTouchHelper.html)。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-core-ui:27.1.1
```

### v4 媒体兼容库

提供了[媒体](https://developer.android.google.cn/reference/android/media/package-summary.html)框架的向后移植，包括 [`MediaBrowser`](https://developer.android.google.cn/reference/android/media/browse/MediaBrowser.html) 和 [`MediaSession`](https://developer.android.google.cn/reference/android/media/session/MediaSession.html)。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-media-compat:27.1.1
```

### v4 fragment 库

提供了 [fragment](https://developer.android.google.cn/guide/components/fragments.html) 的 UI 和功能的封装，为应用提供能适应不同尺寸屏幕的布局。该模块依赖于兼容库、核心实用程序库、核心 UI 库和媒体兼容库。

> **注意**：[v13 支持库](https://developer.android.google.cn/topic/libraries/support-library/packages#v13)提供了一个 [`FragmentCompat`](https://developer.android.google.cn/reference/android/support/v13/app/FragmentCompat.html) 类。该 v4 [`Fragment`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html) 类是一个独立的类，提供了之后几个平台版本更新中添加的漏洞修复，而 v13 [`FragmentCompat`](https://developer.android.google.cn/reference/android/support/v13/app/FragmentCompat.html) 类为框架中的 [`Fragment`](https://developer.android.google.cn/reference/android/app/Fragment.html) 类提供了向后兼容的小型函数库（shim）实现。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-fragment:27.1.1
```

## Multidex 支持库

提供了构建 Dalvik 可执行文件（DEX）分包的应用的支持。方法数超过 65,536 的应用要求使用  Dalvik 可执行文件分包的配置。欲了解更多使用 Multidex 的信息，请参阅[配置方法数超过 64K 的应用](https://developer.android.google.cn/studio/build/multidex)。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:multidex:1.0.0
```

## v7 支持库

下列各库分别提供了某些特性的集合，且能彼此独立地导入您的应用。

### v7 兼容库

This library adds support for the Action Bar user interface design pattern. This library includes support for material design user interface implementations.

Note: This library depends on the v4 Support Library.

Here are a few of the key classes included in the v7 appcompat library:

ActionBar - Provides an implementation of the action bar user interface pattern. For more information on using the Action Bar, see the Action Bar developer guide.
AppCompatActivity - Adds an application activity class that can be used as a base class for activities that use the Support Library action bar implementation.
AppCompatDialog - Adds a dialog class that can be used as a base class for AppCompat themed dialogs.
ShareActionProvider - Adds support for a standardized sharing action (such as email or posting to social applications) that can be included in an action bar.
Note: The appcompat library is an Android Jetpack foundation component. See it in use in the Sunflower demo app.

The Gradle build script dependency identifier for this library is as follows:

com.android.support:appcompat-v7:27.1.1


