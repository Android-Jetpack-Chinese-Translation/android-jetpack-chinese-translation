# 支持库概览
> 原文链接：[Support Library |  Android Developers](https://developer.android.google.cn/topic/libraries/support-library/)

在开发兼容多个 API 版本的应用时，为了在更旧的 Android 版本上提供新的特性、或者至少优雅地达成同样的效果，您可能需要一种标准化的解决方案。与其专门写代码来处理老版本的情况，您不妨利用这些支持库来提供一个兼容层。此外，支持库还额外提供了标准的架构 API 所没有的便利类和特性，用来便利开发流程并兼容更多的设备。

支持库最开始只是应用开发的单个二进制库，但现在已经演化成了整整一套类库——其中不乏我们强烈推荐您使用的库，很多甚至是必不可少的。

本小节对支持库进行了简单的介绍，以便您理解其组件和有效使用的方式。

> **警告**：从支持库版本 26.0.0（2017年7月）开始，大多数支持库所向后兼容的最低 API 版本已提升至 Android 4.0（API 版本 14）。欲了解更多信息，请参阅本小节的 [Version Support and Package Names](https://developer.android.google.cn/topic/libraries/support-library/#api-versions)。

## 支持库的用途

支持库有一些独特的用途，针对早期平台的向后兼容类只是其中之一。如下是较为完整的列表：

* **新 API 的向后兼容性**：许多支持库都提供新的类和方法的向后兼容。例如，支持库里的 [`Fragment`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html) 类给早于 Android 3.0（API 版本 11）版本的设备上提供了 fragment 的支持。
* **便利类和助手类**：支持库提供不少助手类，尤其是 UI 方面的。例如，[`RecyclerView`](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView.html) 类提供了一个用于展示和管理超长列表的 UI 部件，一直向前兼容到 Android API 版本 7。
* **调试和实用程序**：除了您写入应用的代码之外，支持库还提供很多其他的便利特性。例如， [`support-annotations`](https://developer.android.google.cn/studio/write/annotations.html) 类提升了方法输入参数的代码风格检查，而 [Multidex 支持](https://developer.android.google.cn/studio/build/multidex) 可以配置方法数超过 65,536 的应用。

## 支持库对比架构 API

支持库提供的类和方法和 Android 框架里的 API 非常相似，注意到了这一点的话，您可能会疑惑究竟应该使用哪个。这里有几条选用支持库的准则：

* **具体某个特性的兼容性**：如果您想为某个新平台上的某个特性提供向后兼容，请使用支持库里的相应类和方法。
* **相关类库特性的兼容性**：支持库里更复杂一些的类可能依赖于一个或多个额外的支持库类，所以您应当使用支持库的类来满足这些依赖关系。例如，[`ViewPager`](https://developer.android.google.cn/reference/android/support/v4/view/ViewPager.html) 这个支持库的类应当和同样是支持库的 [`FragmentPagerAdapter`](https://developer.android.google.cn/reference/android/support/v4/app/FragmentPagerAdapter.html) 或 [`FragmentStatePagerAdapter`](https://developer.android.google.cn/reference/android/support/v4/app/FragmentStatePagerAdapter.html) 类一起使用。
* **整体的设备兼容性**：即使您并没有想为特定的某个特性提供向后兼容，使用支持库仍然不失为一个好主意：您可以用 [`ActivityCompat`](https://developer.android.google.cn/reference/android/support/v4/app/ActivityCompat.html) 来取代标准的 `Activity` 类，以便利用以后可能加入的一些功能，如 Android 6.0 （API 版本 13）新加入的权限控制。

受宿主设备的平台版本限制，这些实现向后兼容性的支持库类会优雅地把职责降级到旧平台上的实现，但可能无法提供最新版本里的全部功能或数据。在这种情况下，您应当检阅您所使用的类和方法的参考文档，并在您的应用所支持的最旧的设备上全面彻底地测试。

> **注意**：支持库并不为每个 Android 框架 API 都提供相应的类和方法。在某些情况下，您可能需要用一个显式的 SDK 版本检查来确认一个框架 API 调用是否有效，并为不支持该调用的设备提供备选方案。欲了解在您代码中使用版本检查的更多信息，请参阅[兼容不同平台版本](https://developer.android.google.cn/training/basics/supporting-devices/platforms.html)。

## 兼容版本和包名

有些支持库的包名含有 v# 格式的标记，用来显示其最初能兼容到的最老的 API 版本，例如 support-v4。从支持库版本 26.0.0（2017年7月）开始，所有支持库包的向后兼容的最低 API 版本已提升至 Android 4.0（API 版本 14）。因此，当您使用最近发行的支持库时，请不要把这些 v# 标记理解为其支持的最老版本。这个变化也意味着含有 v4 和 v7 标记的支持库向后兼容到的版本在本质上是相同的。例如，对于版本新于 26.0.0 的支持库而言，support-v4 和 support-v7 包都向后兼容到 API 版本 14。

### 支持库的发布版本

支持库的[发布版本](https://developer.android.google.cn/topic/libraries/support-library/revisions.html)（诸如 24.2.0 或 25.0.1）和其所能兼容到的最早的 API 版本是不同的。发布版本表明了该支持库所用来构建的平台版本，因此也就表明了它包含了哪些最新的 API。

特别地，发布版本号码的第一部分（例如 24.2.0 里的 24），一般对应于该支持库发布时可用的平台 API 版本。尽管支持库的发布版本表明它所包含的相应平台 API 的特性，但您不应理解为它为这些新特性全都提供了向后兼容的支持。

## 库依赖

支持库套件里的大部分类库都依赖于至少一个其他的库。例如，几乎所有支持库都依赖于 `support-compat` 包。一般来说，您毋须担心支持库的依赖关系，因为 Gradle 构建工具会为您处理、自动导入所需的库。

如果您想查阅哪些库或库依赖被包含到您的应用中，请在您应用项目的根目录运行如下所示的指令，它会生成该项目的包含了 Android 支持库和其他类库的依赖关系：

```bash
gradle -q dependencies your-app-project:dependencies
```

欲了解使用 Gradle 为您的项目添加支持库的更多信息，请参阅[支持库设置](https://developer.android.google.cn/topic/libraries/support-library/features)*（请选择简体中文）*。欲了解更多使用 Gradle 的信息，请参阅[配置构建](https://developer.android.google.cn/studio/build/)*（请选择简体中文）*。

请注意，*所有* Android 支持库同样依赖于某个特定的平台版本，例如最近发布的支持库需要基于 Android 4.0 （API 版本 14）或更高。

