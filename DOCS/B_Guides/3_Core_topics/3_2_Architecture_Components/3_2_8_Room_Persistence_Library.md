# Room 数据持久化库
> 原文链接：[Room Persistence Library  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/room)

[Room](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_8_App_data_&_files/3_8_5_Save_data_in_a_local_database/3_8_5_1_Overview.md) 数据持久化库提供了一个基于 SQLite 的抽象层，以便在利用 SQLite 全部功能的同时实现更稳健的数据库访问。

该库帮您在设备上创建您应用数据的缓存，并让该缓存充当您应用的唯一数据来源，从而让用户能访问到关键信息的一致性副本，无论他是否有网络连接。

> **注意**：欲将 Room 导入您的 Android 项目，请参阅 [Room 发布说明](https://developer.android.google.cn/jetpack/androidx/releases/room)。

## 深入文档

有关将 Room 的功能运用于程序数据持久化存储的指导方案，请参阅 [Room](https://developer.android.google.cn/training/data-storage/room/index.html) 培训指南。

## 其他资源

要了解有关 Room 的更多信息，请参阅以下其他资源。

### 示例
- [Sunflower](https://github.com/googlesamples/android-sunflower), 一款 Android Jetpack 最佳实践的示例应用。
- [Room migration sample](https://github.com/googlesamples/android-architecture-components/tree/master/PersistenceMigrationsSample)（Room 迁移示例）
- Room & RxJava Sample [(Java)](https://github.com/googlesamples/android-architecture-components/tree/master/BasicRxJavaSample) [(Kotlin)](https://github.com/googlesamples/android-architecture-components/tree/master/BasicRxJavaSampleKotlin)

### Codelabs
- Android Room with a View [(Java)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view) [(Kotlin)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view-kotlin)
- [Android Persistence codelab](https://codelabs.developers.google.com/codelabs/android-persistence/index.html?index=..%2F..%2Findex#0)

### 博客
- [Introducing Android Sunflower](https://medium.com/androiddevelopers/introducing-android-sunflower-e421b43fe0c2)
- [Room + Time](https://medium.com/androiddevelopers/room-time-2b4cf9672b98?source=false---------7)
- [Incrementally migrate from SQLite to Room](https://medium.com/androiddevelopers/incrementally-migrate-from-sqlite-to-room-66c2f655b377)
- [7 Pro-tips for Room](https://medium.com/androiddevelopers/7-pro-tips-for-room-fbadea4bfbd1)
- [Understanding migrations with Room](https://medium.com/androiddevelopers/understanding-migrations-with-room-f01e04b07929)
- [Testing Room migrations](https://medium.com/androiddevelopers/testing-room-migrations-be93cdb0d975)
- [Room + RxJava](https://medium.com/androiddevelopers/room-rxjava-acb0cd4f3757)
