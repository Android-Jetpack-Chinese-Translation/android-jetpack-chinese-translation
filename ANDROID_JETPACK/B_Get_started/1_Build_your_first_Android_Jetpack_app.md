# “Hello, Jetpack”：开发您的第一个 Jetpack 应用
> 原文链接：["Hello, Jetpack": Build your first Jetpack app  |  Android Developers](https://developer.android.google.cn/jetpack/docs/getting-started)

本章节将通过一个叫做 [ComponentsBasicSample](https://developer.android.google.cn/jetpack/docs/ComponentsBasicSample.zip) 的 Kotlin 案例应用，来向您展示如何开发一个基于 Jetpack 的简单应用。

欲了解如何开发一般的应用，请参考[构建您的第一个应用](https://developer.android.google.cn/training/basics/firstapp/)。欲详细了解本章节所使用的类（class），请参阅 [Android 应用架构组件](https://developer.android.google.cn/jetpack/arch/)。

## 搭建您的应用
按照以下步骤来搭建您的应用，并为其提供 Jetpack 支持。

1. 启动3.2或更高版本的 [Android Studio](https://developer.android.google.cn/studio/preview)，照常选择 ***Create Android Project*** 、 ***Target Android Devices*** 并填入所需信息。

    > 如果您打算使用 Kotlin 来开发应用，请记得在 ***Create Android Project*** 页面勾选 ***Include Kotlin Support*** 。

2. ***Create Android Project*** 的下一个页面是 ***Add an Activity to Mobile*** ，该页面提供了许多用于创建应用的模板。如图一所示的 ***Activity & Fragment + ViewModel*** 模板能够方便地将 Jetpack 整合进您的应用，选择它并点击 ***Next*** 。
    
    ![图一：Activity & Fragment + ViewModel 模板](https://developer.android.google.cn/images/jetpack/gs-1.png)

3. 在 ***Configure Activity*** 页面，输入应用初始 Activity、Fragment 和 ViewModel 的名称。您也可以选择性地输入 fragment 的 package 路径。之后点击 ***Finish*** 。
    ![图二：Activity & Fragment + ViewModel 模板的 Configure Activity 页面](https://developer.android.google.cn/images/jetpack/gs-2.png)

在您的项目（Project）视图中打开 `java` 文件夹，如图三所示，应用初始即含有三个类：`StartActivity`，`StartFragment` 和 `StartViewModel`。
    ![图三：Activity & Fragment + ViewModel 模板初始化的类](https://developer.android.google.cn/images/jetpack/gs-3.png)

- `StartActivity` 是您应用的入口。它是一个 [`Activity`](https://developer.android.google.cn/reference/android/app/Activity) 的桩（Stub），用来充当您应用初始界面的 Fragment 的容器。
- `StartFragment` 是您初始界面的 [`Fragment`](https://developer.android.google.cn/reference/android/app/Fragment) 的桩。
- `StartViewModel` 是您初始 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel) 的桩。

## 利用 Jetpack 的优势
上述组件准备就绪，您可以选择按如下的例子所示来实现您的 ViewModel 对象：

```Kotlin
class StartViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String>
        get() = _data

    init {
        _data.value = "Hello, Jetpack!"
    }
}
```

如果您的应用包含不止一个页面，您可以导入 [`Navigation`](https://developer.android.google.cn/topic/libraries/architecture/navigation) 类来实现 Fragment 的导航。下面的代码示范了如何实现导航逻辑：

```Kotlin
// Set up a click listener on the login button
view?.findViewById<Button>(R.id.navigate_bt)?.setOnClickListener {
   // Navigate to the login destination
   view?.let { Navigation.findNavController(it).navigate(R.id.end_action) }
```

如果您的应用需要接入 SQLite 数据库，您还可以导入数据持久化类库 [`Room`](https://developer.android.google.cn/reference/android/arch/persistence/room/Room)。如果您的应用需要在单个页面上显示大量数据，您应该考虑使用 [`Paging`](https://developer.android.google.cn/topic/libraries/architecture/paging) 类库。

## 配置您的 Gradle 脚本
为了使用 Jetpack，您必须牢记在 Gradle 脚本中添加相应的语句。本案例使用了 `ViewModel`，`LiveData`，`NavigationController`，因此其 Gradle 脚本中含有这些语句：

```Gradle
// LiveData + ViewModel
implementation "android.arch.lifecycle:extensions:$rootProject.archLifecycleVersion"

// Navigation
implementation 'androidx.navigation:navigation-fragment:' + rootProject.navigationVersion
implementation 'androidx.navigation:navigation-ui:' + rootProject.navigationVersion
```

## 其他资源

想要了解更多案例和本章节涉及的类，请参阅以下资源：
- [Android 应用架构组件](https://developer.android.google.cn/jetpack/arch/)
- [ComponentsBasicSample](https://developer.android.google.cn/jetpack/docs/ComponentsBasicSample.zip)

