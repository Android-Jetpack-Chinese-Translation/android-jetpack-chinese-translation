# 迁移到 Navigation 架构组件
> 原文链接：[Migrate to the Navigation Architecture Component  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation/navigation-migrate)

[`NavController`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn) 及其导航图是包含在单个 activity 中的。因此，当您把一个现有的项目迁移到 Navigation 架构组件上时，请每次只集中精力在迁移一个 activity 上，为每个 activity 中的目的地创建导航图。

![图一：Activity 及其各自的导航图](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-migrate1.png?hl=zh-cn)
**图一：**Activity 及其各自的导航图

各个 activity 可以通过向其导航图添加 activity 目的地的方式来连接其他的 activity，用于取代整个代码库中的 [`startActivity()`](https://developer.android.google.cn/reference/android/app/Activity?hl=zh-cn#startactivity) 调用。

![图二：Activity A 中的导航图指向 Activity B](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-migrate2.png?hl=zh-cn)
**图二：**Activity A 中的导航图指向 Activity B

在多个 activity 共用相同布局的情况下，导航图可以被合并，并将导航到 activity 目的地的 `navigate()` 调用替换成两个导航图之间的直接导航。

![图三：含有合并导航图的 activity](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-migrate3.png?hl=zh-cn)
**图三：**含有合并导航图的 activity


