# Android 架构组件
> 原文链接：[Android Architecture Components  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/)

Android 架构组件是一组帮助您构建具有稳健性、可测试性、可维护性应用的类库。先从如下的两个话题开始吧：1、管理 UI 组件生命周期，2、处理数据持久化。

已推出 1.0 稳定版。

#### [开始动手](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/D_Libraries/2_Architecture_Components/2_2_Adding_Components_to_your_Project.md)

#### [观看视频](https://www.youtube.com/watch?v=vOJCrbr144o)

## 轻松管理您应用的生命周期

生命周期感知（lifecycle-aware）的新的组件能帮您管理 activity 和 fragment 的生命周期：使用 [LiveData](https://developer.android.google.cn/topic/libraries/architecture/livedata)、[ViewModel](https://developer.android.google.cn/topic/libraries/architecture/viewmodel)、[LifecycleObserver](https://developer.android.google.cn/topic/libraries/architecture/lifecycle)、[LifecycleOwner](https://developer.android.google.cn/topic/libraries/architecture/lifecycle)，您的应用能够平稳渡过配置变更（configuration change）、避免内存泄漏，还能轻松将数据载入 UI。

#### [试试这个 Codelab](https://codelabs.developers.google.com/codelabs/android-lifecycles/#0)

#### [查阅文档](https://developer.android.google.cn/topic/libraries/architecture/lifecycle)

#### [获取项目范例](https://github.com/googlesamples/android-architecture-components)

## Room：一个 SQLite 对象关系映射类库

使用 [Room](https://developer.android.google.cn/topic/libraries/architecture/room) 来避免八股代码，并将 SQLite 数据表的数据映射为 Java 对象。Room 能对 SQLite 语句进行编译时检查，并返回 RxJava、Flowable 和 LiveData 的可观测对象.

#### [试试这个 Codelab](https://codelabs.developers.google.com/codelabs/android-persistence/#0)

#### [查阅文档](https://developer.android.google.cn/topic/libraries/architecture/room)

#### [获取项目范例](https://github.com/googlesamples/android-architecture-components)




