# Android 架构组件
> 原文链接：[Android Architecture Components  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/)

Android 架构组件是一组帮助您构建具有稳健性、可测试性、可维护性应用的类库。先从如下的两个话题开始吧：1、管理 UI 组件生命周期，2、处理数据持久化。

- 轻松管理您的应用生命周期。新的 [生命周期感知（lifecycle-aware）组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_4_Handling_Lifecycles.md) 能帮您管理 activity 和 fragment 的生命周期。您的应用能够平稳渡过配置变更（configuration change）、避免内存泄漏，还能轻松将数据载入 UI。
- 使用 [LiveData](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_5_LiveData.md) 构建数据对象，以便在底层数据库变动时通知视图。
- [ViewModel](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_9_ViewModel.md) 可在应用旋转屏幕时依然保持和 UI 相关的数据。
- [Room](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md) 是一个 SQLite 对象关系映射类库。用它来避免八股代码，并将 SQLite 数据表的数据映射为 Java 对象。Room 能对 SQLite 语句进行编译时检查，并返回 RxJava、Flowable 和 LiveData 的可观测对象。

## 其他资源

欲知更多有关架构组件的信息，请参阅下列资源。

### 示例
- [Sunflower](https://github.com/googlesamples/android-sunflower)，一款用 Android Jetpack 来进行最佳实践开发演示的园艺应用。
- [Android Architecture Components basic sample](https://github.com/googlesamples/android-architecture-components/tree/master/BasicSample)
- [(更多...)](https://developer.android.google.cn/topic/libraries/architecture/additional-resources.html#samples)

### Codelabs（代码实验室）
- Android Room with a View [(Java)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view) [(Kotlin)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view-kotlin)（Room 和视图层结合）
- [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)（数据绑定代码示例）
- [(更多...)](https://developer.android.google.cn/topic/libraries/architecture/additional-resources.html#codelabs)


### 培训
- [Udacity: Developing Android Apps](https://www.udacity.com/course/new-android-fundamentals--ud851)（优达城：开发 Android 应用的培训）

### 博客文章
- [Announcing Architecture Components 1.0 Stable](https://android-developers.googleblog.com/2017/11/announcing-architecture-components-10.html)（Android 架构组件 1.0 稳定版）
- [Android and Architecture](https://android-developers.googleblog.com/2017/05/android-and-architecture.html)（Android 架构）
- [(更多...)](https://developer.android.google.cn/topic/libraries/architecture/additional-resources.html#blogs)

### 视频
- [Android Jetpack: what's new in Architecture Components (Google I/O '18)](https://www.youtube.com/watch?v=pErTyQpA390)（Android Jetpack：架构组件新特性（Google 2018 开发者大会））
- [Android Jetpack: easy background processing with WorkManager (Google I/O '18)](https://www.youtube.com/watch?v=IrKoBFLwTN0)（Android Jetpack：用 WorkManager 轻松处理后台任务（Google 2018 开发者大会））
- [(更多...)](https://developer.android.google.cn/topic/libraries/architecture/additional-resources.html#videos)
