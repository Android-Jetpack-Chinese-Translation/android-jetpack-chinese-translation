# 使用 WorkManager 来计划任务
> 原文链接：[Schedule tasks with WorkManager  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/workmanager/)

WorkManager 的 API 让您能轻松地指定可推迟的异步任务并安排其执行的时间。您只需创建一个任务并将其交给 WorkManager，就可以立即或在合适的时间执行它。

基于设备的 API 版本和应用的状态等因素，WorkManager 可为您的任务选择合适的执行方式。如果 WorkManager 在您的应用仍在运行的时候执行一个任务，那么它会为这个任务创建一个新的线程；如果您的应用不在运行，那么它会根据设备的 API 版本及所包含的依赖关系来安排一个后台任务，包括 [`JobScheduler`](https://developer.android.google.cn/reference/android/app/job/JobScheduler.html)、[`Firebase JobDispatcher`](https://github.com/firebase/firebase-jobdispatcher-android#user-content-firebase-jobdispatcher-) 和 [`AlarmManager`](https://developer.android.google.cn/reference/android/app/AlarmManager.html)。无须手动编写检查设备能力的代码来选择合适的 API，您只需把任务交给 WorkManager 来让其自主选择最佳方案。

> **注意**：WorkManager 是专门为即使应用退出也必须保证系统执行的任务而设计的，例如将应用数据上传到服务器。它并不适用于即使应用退出也能被安全终止的进程内的后台任务，针对这种情况，我们推荐您使用 [`ThreadPools`](https://developer.android.google.cn/training/multiple-threads/create-threadpool#ThreadPool)。

## 话题

#### [WorkManager 基础](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_10_WorkManager/3_2_10_2_Basics.md)

使用 WorkManager 来安排单个任务在您指定的情况下执行，或者每隔特定时间间隔就执行的重复任务。

#### [WorkManager 进阶话题](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_10_WorkManager/3_2_10_3_Advanced.md)

设定任务的链式序列（chained sequences），设定可以传递并返回值的任务，设定命名的、独特的任务序列。


