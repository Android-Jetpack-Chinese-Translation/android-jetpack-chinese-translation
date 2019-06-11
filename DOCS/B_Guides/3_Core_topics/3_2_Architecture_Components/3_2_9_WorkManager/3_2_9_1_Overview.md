# 使用 WorkManager 来调度任务
> 原文链接：[Schedule tasks with WorkManager  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/workmanager/)

WorkManager 的 API 让您能轻松地指定可推迟的异步任务并安排其执行的时间，即使应用已经退出或者设备重启。

**主要特点：**
- 向后兼容至 API 14
    - 在 API 23+ 的设备上使用 JobScheduler
    - 在 API 14-22 的设备上组合使用 BroadcastReceiver + AlarmManager
- 添加网络可用性或电量等工作限制
- 调度一次性或周期性的异步任务
- 监测和管理已调度的任务
- 任务可连锁在一起
- 即使应用程序或设备重新启动，也可保证任务执行

WorkManager 适用于可推迟的、无需立即执行的任务，即使应用程序退出或设备重启也要保证可靠运行。例如:
- 发送日志或分析数据给服务端
- 定期与服务器同步应用程序数据

WorkManager 不适用于在应用进程消失时可以安全终止的进程内的后台任务，或者需要立即执行的任务。请参阅 [后台任务指南](https://developer.android.google.cn/guide/background/) 以明确您需要哪种解决方案。

查看 [发行说明](https://developer.android.google.cn/jetpack/androidx/releases/work#declaring_dependencies) 以了解如何将 WorkManager 导入 Android 项目。

## 话题

#### [WorkManager 基础知识](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_9_WorkManager/3_2_9_2_Basics.md)

使用 WorkManager 来调度特定条件下执行的单次任务，或者是定期运行的任务。

#### [WorkManager 高级进阶](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_9_WorkManager/3_2_9_3_Advanced.md)

设置任务的链式序列（chained sequences），设置可传参并返回值的任务，设置指定的、唯一的任务序列。

#### [从 Firebase JobDispatcher 迁移](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_9_WorkManager/3_2_9_4_Migrating_from_Firebase_JobDispatcher.md)

更新现有应用，以便使用 WorkManager 而不是 Firebase JobDispatcher。

## 更多资源

### 示例

- [WorkManagerSample](https://github.com/googlesamples/android-architecture-components/tree/master/WorkManagerSample)，一款简单的图像处理应用
- [Sunflower](https://github.com/googlesamples/android-sunflower)，一款演示各种架构组件用法的示例应用，包括 WorkManager。

### Codelabs

- 使用 WorkManager，支持 [Kotlin](https://codelabs.developers.google.com/codelabs/android-workmanager-kt/) 和 [Java](https://codelabs.developers.google.com/codelabs/android-workmanager/) 代码

### 视频

- [使用 WorkManager](https://www.youtube.com/watch?v=83a4rYXsDs0)，来源：2018 Android 开发者峰会

### 博客

- [介绍 WorkManager](https://medium.com/androiddevelopers/introducing-workmanager-2083bcfc4712)
