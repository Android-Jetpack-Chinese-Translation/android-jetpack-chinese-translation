# Android Jetpack
> 原文链接：[Android Jetpack  |  Android Developers](https://developer.android.google.cn/jetpack/)

Jetpack 是一组包含了类库、工具和架构指导的集合，用来协助您轻松构建出色的 Android 应用。由于它已经提供了常用的基础框架，您可以集中精力开发您的应用所特有的业务代码。

[开始](https://developer.android.google.cn/jetpack/docs/getting-started/)

[观看介绍视频](https://www.youtube.com/watch?v=LmkKFCfmnhQ&feature=youtu.be)

* 加速开发
    
    Jetpack 的诸多组件（component）可以分别独自使用，而且作为提升您生产力的有机统一的整体，在 Kotlin 编程语言的助力下更是如虎添翼。
    
* 去除“八股代码”（boilerplate code）
    
    Jetpack 已为您一一打理好各种枯燥乏味的业务，诸如：后台任务、导航（navigation）和生命周期管理（lifecycle management），等等。
    
* 构建优质而稳健的应用
    
    遵循现代设计规范，Jetpack 的组件减少了应用崩溃，降低了内存占用，还支持向后兼容（backwards compatibility）。
    
## Android Jetpack 组件（component）

Jetpack 组件是一组类库的集合，它们可以分别独自使用，而且作为提升您生产力的有机统一的整体，在 Kotlin 编程语言的助力下更是如虎添翼。是将其全员出场还是自由混搭，您说了算！

### 地基（Foundation）

地基组件提供核心的系统功能、Kotlin 拓展、Dalvik 可执行文件分包（multidex）支持和自动化测试。

* [**AppCompat**](https://developer.android.google.cn/topic/libraries/support-library/packages.html#v7-appcompat)
    
    优雅地向下兼容到更老的 Android 版本。
    
* [**Android KTX**](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/4_Android_KTX.md)
    
    编写更精炼、更地道的 Kotlin 代码。
    
* [**Dalvik 可执行文件分包**](https://developer.android.google.cn/studio/build/multidex)（请选择简体中文）

    为应用提供 Dalvik 可执行文件分包支持。
    
* [**测试支持库**](https://developer.android.google.cn/topic/libraries/testing-support-library)（请选择简体中文）
    
    一套 Android 自动测试框架，用于单元测试和运行时 UI 测试。
    
### 架构（Architecture）
    
架构组件帮助您管理 UI 组件的生命周期，处理数据持久化（data persistence），等等。
    
* **数据绑定（Data binding）**

    声明式（declaratively）地将可被监听的数据（observable data）绑定到 UI 元素。
    
* **生命周期**
    
    管理您的 activity 和 fragment 的生命周期。
    
* **LiveData**
    
    当其来源的数据库发生变化时，能够自动通知视图组件（view）。
    
* **导航（Navigation）**

    应用内导航的一切要素。

* **分页（Paging）**
    
    按需求逐步地从您的数据源加载信息。
    
* **Room**
    
    流畅的 SQLite 数据库连接。
    
* **视图模型（ViewModel）**

    用一种生命周期感知（lifecycle-conscious）的方式来管理 UI 相关的数据。
    
* **后台任务管家（WorkManager）**
    
    管理您的 Android 后台任务。

### 行为（Behavior）

行为组件帮助您设计稳健、可测试且可维护的应用。
    
* **下载管家（Download manager）**
    
    安排并管理大文件的下载。
    
* **媒体和回放（Media & playback）**
    
    向后兼容的用于媒体回放和路由（routing）的API，包括 Google Cast。
    
* **通知 （Notification）**
    
    向后兼容并支持可穿戴设备（Wear）和驾驶设备（Auto）的通知 API。
    
* **权限（Permissions）**
    
    用于检查和请求应用权限的兼容性 API。
    
* **分享（Sharing）**
    
    提供一个适用于应用操作栏（action bar）的分享操作（action）。
    
* **切片（Slices）**

    创建灵活的 UI 元素，用于在应用之外展示其数据。

### 交互（UI）

交互组件使您轻易构造出易于上手、体验美妙的应用。
    
* **动画和渐变（Animation & transitions）**
    
    将部件（widget）和渐变跨页面展示。
    
* **驾驶设备（Auto）**
    
    帮助您为驾驶设备开发应用的组件。
    
* **绘文字（Emoji）**
    
    在旧设备上也能使用最新潮流的绘文字。
    
* **Fragment**
    
    可组合 UI 设计（composable UI）的一个基本单元。
    
* **调色板（Palette）**
    
    从色彩组合中提取出有用的信息。
    
* **电视（TV）**

    帮助您为电视开发应用的组件。
    
* Google的**可穿戴设备操作系统（Wear OS）**
    
    帮助您为可穿戴设备开发应用的组件。
    
    

