# 支持库软件包
> 原文链接：[Support Library Packages  |  Android Developers](https://developer.android.google.cn/topic/libraries/support-library/packages)

Android 支持库含有若干可导入到您的应用的软件包，它们各自支持一定特定范围内的 Android 平台版本和功能特性。

欲使用下列任意一个库，您必须下载库文件到您的 SDK 安装目录，按照[支持库设置](https://developer.android.google.cn/tools/support-library/setup.html#download)*（请选择简体中文）*的指导进行操作即可。在这之后，您还必须将特定的支持库导入到您的应用中，导入每个库的具体方法请参阅相应段落的结尾。

> **注意**：所有支持库软件包所支持的最低 SDK 版本至少是 API 版本 14，有些软件包要求的更高，下面会在注意事项中一一提及。

## v4 支持（Support）库

下列各库相对于其他库而言包含了最多的 API，提供了对应用组件、UI 特性、无障碍、数据处理、网络连接和编程实用程序的支持。

欲获取 v4 支持库提供的类和方法的完整详细信息，请参阅 API 文档中的 [`android.support.v4`](https://developer.android.google.cn/reference/android/support/v4/app/package-summary.html) 软件包。

> **注意**：在支持库修订版本 24.2.0 之前，曾只有一个单独的 v4 支持库，该库后来被分割成若干模块来提升效率。如果您在 Gradle 脚本中列出了 **support-v4**，您生成的 APK 安装包将包含所有 v4 的模块以向后兼容。然而，我们仍然推荐您只列出您应用所需的特定模块，以便减小 APK 安装包的大小。

### v4 compat 库

为若干 Android 框架 API 提供向后兼容的包装类，例如 `Context.obtainDrawable()` 和 `View.performAccessibilityAction()`。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-compat:28.0.0
```

### v4 core-utils 库

提供了若干实用类，例如 [`AsyncTaskLoader`](https://developer.android.google.cn/reference/android/support/v4/content/AsyncTaskLoader.html) 和 [`PermissionChecker`](https://developer.android.google.cn/reference/android/support/v4/content/PermissionChecker.html)

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-core-utils:28.0.0
```

### v4 core-ui 库

实现了许多和 UI 相关的组件，例如 [`ViewPager`](https://developer.android.google.cn/reference/android/support/v4/view/ViewPager.html)、[`NestedScrollView`](https://developer.android.google.cn/reference/android/support/v4/widget/NestedScrollView.html) 和 [`ExploreByTouchHelper`](https://developer.android.google.cn/reference/android/support/v4/widget/ExploreByTouchHelper.html)。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-core-ui:28.0.0
```

### v4 media-compat 库

提供了[媒体（Media）](https://developer.android.google.cn/reference/android/media/package-summary.html)框架的向后移植，包括 [`MediaBrowser`](https://developer.android.google.cn/reference/android/media/browse/MediaBrowser.html) 和 [`MediaSession`](https://developer.android.google.cn/reference/android/media/session/MediaSession.html)。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-media-compat:28.0.0
```

### v4 fragment 库

提供了 [fragment](https://developer.android.google.cn/guide/components/fragments.html) 的 UI 和功能的封装，为应用提供能适应不同尺寸屏幕的布局。该模块依赖于兼容库、核心实用程序库、核心 UI 库和媒体兼容库。

> **注意**：[v13 支持库](https://developer.android.google.cn/topic/libraries/support-library/packages#v13)提供了一个 [`FragmentCompat`](https://developer.android.google.cn/reference/android/support/v13/app/FragmentCompat.html) 类。该 v4 [`Fragment`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html) 类是一个独立的类，提供了之后几个平台版本更新中添加的漏洞修复，而 v13 [`FragmentCompat`](https://developer.android.google.cn/reference/android/support/v13/app/FragmentCompat.html) 类为框架中的 [`Fragment`](https://developer.android.google.cn/reference/android/app/Fragment.html) 类提供了向后兼容的小型函数库（shim）实现。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:support-fragment:28.0.0
```

## Multidex 支持库

提供了构建 Dalvik 可执行文件（DEX）分包的应用的支持。方法数超过 65,536 的应用要求使用  Dalvik 可执行文件分包的配置。欲了解更多使用 Multidex 的信息，请参阅[配置方法数超过 64K 的应用](https://developer.android.google.cn/studio/build/multidex)。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:multidex:1.0.0
```

## v7 支持（Support）库

下列各库分别提供了某些特性的集合，且能彼此独立地导入您的应用。

### v7 appcompat 库

提供了对于应用操作栏（action bar）的 UI 设计样式，以及 Material Design 的 UI 实现的支持。

> **注意**：该类库依赖于 v4 支持库

如下是 v7 兼容库所含的若干关键类：

* [`ActionBar`](https://developer.android.google.cn/reference/android/support/v7/app/ActionBar.html)：提供了应用操作栏的 UI 样式。欲了解使用应用操作栏的更多信息，请参阅[应用操作栏](https://developer.android.google.cn/guide/topics/ui/actionbar.html)文档。
* [`AppCompatActivity`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html)：添加了一个应用 activity 类，可充当使用了支持库应用操作栏的activity的基类。
* [`AppCompatDialog`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatDialog.html)：添加了一个对话框类，可充当使用了 AppCompat 主题的对话框的基类。
* [`ShareActionProvider`](https://developer.android.google.cn/reference/android/support/v7/widget/ShareActionProvider.html)：添加了对于可用在应用操作栏的标准化的分享操作（例如邮件或发布到社交网络）的支持。

> **注意**：v7 appcompat 库是 [Android Jetpack](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/A_Overview.md) 的地基组件（foundation component）之一。请参阅 Sunflower 示例应用。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:appcompat-v7:28.0.0
```

### v7 cardview 库

提供了对于 [`CardView`](https://developer.android.google.cn/reference/android/support/v7/widget/CardView.html) 部件的支持，从而让您在不同的应用上都能用外观一致的卡片来展示信息。这些卡片在 Material Design 的实现中很重要，并且还广泛用于 TV 应用中的布局。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:cardview-v7:28.0.0
```

### v7 gridlayout 库

当您下载了 Android 支持库后，v7 gridlayout 库提供了 [`GridLayout`](https://developer.android.google.cn/reference/android/support/v7/widget/GridLayout.html) 的支持，它允许您用矩形单元格组成的表格来安排 UI 元素。欲了解详细内容，请参阅 API 文档中的 [`android.support.v7.widget`](https://developer.android.google.cn/reference/android/support/v7/widget/package-summary.html) 包。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:gridlayout-v7:28.0.0
```

### v7 mediarouter 库

该库提供了对于 [`MediaRouter`](https://developer.android.google.cn/reference/android/support/v7/media/MediaRouter.html)、[`MediaRouteProvider`](https://developer.android.google.cn/reference/android/support/v7/media/MediaRouteProvider.html) 及其相关的兼容 [Google Cast](https://developers.google.cn/cast/) 的媒体类的支持。

总体而言，v7 mediarouter 库的 API 提供了一种控制媒体频道的路由和从当前设备串流到外部屏幕、音响和其他目标设备的途径，包括：发布与当前应用相关的媒体路由提供者、发现并选择目标设备、检查媒体状态，等等。欲了解详细内容，请参阅 API 文档中的 [`android.support.v7.media`](android.support.v7.media) 包。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:mediarouter-v7:28.0.0
```

> 修订版本18的支持库里引入的 v7 mediarouter 库 API 有可能会在更新的版本里改动。当前，我们建议您仅在连接 [Google Cast](https://developers.google.cn/cast/) 时使用该库。

### v7 palette 库

提供了对于 [`Palette`](https://developer.android.google.cn/reference/android/support/v7/graphics/Palette.html) 的支持，它允许您从图像中提取关键颜色。例如，一个音乐应用可以使用一个 [`Palette`](https://developer.android.google.cn/reference/android/support/v7/graphics/Palette.html) 对象来从专辑封面中提取主题色，并使用它们来构建颜色相称的歌曲标题卡片。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:palette-v7:28.0.0
```

### v7 recyclerview 库

添加了 [`RecyclerView`](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView.html) 类，它提供了对 [`RecyclerView`](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView.html)  部件的支持。该类通过一个包含了有限数量元素的窗口，能够高效地展示庞大的数据集合。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:recyclerview-v7:28.0.0
```

### v7 Preference 支持库

[`preference`](https://developer.android.google.cn/reference/android/support/v7/preference/package-summary) 包提供了对于添加 UI 设置的偏好设定对象（例如 [` CheckBoxPreference`](https://developer.android.google.cn/reference/android/support/v7/preference/CheckBoxPreference.html) 和 [`ListPreference`](https://developer.android.google.cn/reference/android/support/v7/preference/ListPreference.html)）、接口 [`Preference.OnPreferenceChangeListener`](https://developer.android.google.cn/reference/android/support/v7/preference/Preference.OnPreferenceChangeListener.html) 和 [`Preference.OnPreferenceClickListener`](https://developer.android.google.cn/reference/android/support/v7/preference/Preference.OnPreferenceClickListener.html)  的支持。

在 Gradle 构建脚本中添加对该库的依赖所需的识别符是：

```gradle
com.android.support:preference-v7:28.0.0
```

