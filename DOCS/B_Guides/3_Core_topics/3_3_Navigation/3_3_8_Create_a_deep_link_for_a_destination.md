# 为目的地创建深层链接
> 原文链接：[Create a deep link for a destination  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation/navigation-deep-link)

在 Android 中，**深层链接（Deep link）** 是能够直接把用户带到应用中特定目的地的链接。

您可以使用 Navigation 组件创建两种不同类型的深层链接：显式（explicit）深层链接、隐式（implicit）深层链接。

## 创建一个显式深层链接

[显式深层链接](https://developer.android.google.cn/training/app-links/deep-linking?hl=zh-cn)是一个使用 [`PendingIntent`](https://developer.android.google.cn/reference/android/app/PendingIntent?hl=zh-cn) 来将用户带到您应用中某个特定位置的深层链接实例。您可以把显式深层链接做成通知、捷径（app shortcut）、小部件（widget）等的一部分。

当用户使用显式深层链接打开您的应用时，任务回退栈被清空并被替换成该深层链接指向的目的地。当[嵌套导航图](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_6_Navigation/3_2_6_4_Nested_graphs.md)存在时，每层嵌套的起始目的地（即每个 `<navigation>` 元素的层级上的起始目的地）也会被加入回退栈。这意味着，当用户在该深层链接指向的目的地上点击”返回“按键时，他/她会回到导航栈中的上一层，就和从应用入口一层层导航到该目的地的情形一样。

您可以使用 [`NavDeepLinkBuilder`](https://developer.android.google.cn/reference/androidx/navigation/NavDeepLinkBuilder?hl=zh-cn) 类来构造一个 [`PendingIntent`](https://developer.android.google.cn/reference/android/app/PendingIntent?hl=zh-cn)，如下例所示。请注意，如果提供的 `Context` 不是 `Activity`，那么如果可以的话，构造器就会使用 [`PackageManager.getLaunchIntentForPackage()`](https://developer.android.google.cn/reference/android/content/pm/PackageManager?hl=zh-cn#getLaunchIntentForPackage(java.lang.String)) 来作为默认的启动 activity。

```java
PendingIntent pendingIntent = new NavDeepLinkBuilder(context)
    .setGraph(R.navigation.nav_graph)
    .setDestination(R.id.android)
    .setArguments(args)
    .createPendingIntent();
```

如果您手头有一个 [`NavController`](https://developer.android.google.cn/reference/androidx/navigation/NavController?hl=zh-cn)，那么您还能通过 [`NavController.createDeepLink()`](https://developer.android.google.cn/reference/androidx/navigation/NavController?hl=zh-cn#createDeepLink())。

## 创建一个隐式深层链接

[隐式深层链接](https://developer.android.google.cn/training/app-links/deep-linking?hl=zh-cn) 是一个用来引用到您应用中特定目的地的 URI。当一个 URI 被唤起（例如当用户点击某个链接）时，Android 就会接着打开您应用中的相应目的地。

当一个隐式深层链接被触发时，回退栈的状态依赖于 `Intent` 是否使用 [`Intent.FLAG_ACTIVITY_NEW_TASK`](https://developer.android.google.cn/reference/android/content/Intent?hl=zh-cn#FLAG_ACTIVITY_NEW_TASK) 的旗标：

* 设置旗标时，回退栈会被清空，并被替换成深层链接指向的目的地。如上小节中的显示深层链接所述。
* 不设置旗标时，用户仍留在之前触发深层链接的应用的回退栈上。在这种情况下，”返回“按钮会把用户带回前一个应用，而“向上”按钮会启动您应用的导航图层级结构中的上层目的地。

按如下指示操作，您可以使用导航编辑器来创建一个指向特定目的地的隐式深层连接：

1. 在导航编辑器中的 **Design** 标签页中，选择您欲为深层链接添加的目的地。
2. 单击 **Attributes** 面板中的 **Deep Links** 部分的 **+**。
3. 在新出现的 **Add Deep Link** 对话框中，输入一个 **URI**。请注意：
	* 没有协议名（scheme）的 URI 会被认为是 “http” 或 “https”。例如，`www.google.com` 会被同时匹配到 `http://www.google.com` 和 `https://www.google.com`。
	* 以 `{placeholder_name}` 格式的占位符可以匹配一个或多个字符。例如，`http://www.example.com/users/{id}` 可以匹配到 `http://www.example.com/users/4`。导航组件会试图通过将占位符的名字匹配到目的地所定义的[数据参数](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_6_Navigation/3_2_6_5_Pass_data_between_destinations.md)的方式，把占位符解析成合适的类型。
	* 您可以使用 `.*` 通配符来匹配零或多个字符。
5. （可选）勾选 **Auto Verify** 来要求谷歌验证您是否是该 URI 的所有者。欲了解更多信息，请参阅[验证 Android 应用链接](https://developer.android.google.cn/training/app-links/verify-site-associations.html?hl=zh-cn)。
6. 单击 **Add**，一个链接图标![](https://developer.android.google.cn/studio/images/buttons/navigation-deeplink.png?hl=zh-cn)会在所选的目的地上面出现，用以表明该目的地拥有深层链接。
7. 单击 **Text** 标签页来切换到 XML 视图。一个嵌套的 `<deeplink>` 元素就会被加入到所选的目的地：

	```xml
	<deepLink app:uri="https://www.google.com" />
	```

欲启用隐式深层链接，您必须也为您应用的 `manifest.xml` 文件额外添加内容：为指向现有导航图的 activity 添加一个 `<nav-graph>` 元素，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapplication">

    <application ... >

        <activity name=".MainActivity" ...>
            ...

            <nav-graph android:value="@navigation/nav_graph" />

            ...

        </activity>
    </application>
</manifest>
```

当您构建项目时，Navigation 组件会将 `<nav-graph>` 元素替换为自动生成的 `<intent-filter>` 元素来匹配导航图中所有的深层链接。

> **注意**：Android Studio 3.0 和 3.1 并不支持 `<nav-graph>` 元素。当使用这些版本时，您必须手动添加 `<intent-filter>` 元素。欲了解更多信息，请参阅 [为应用内容创建深层链接](https://developer.android.google.cn/training/app-links/deep-linking.html?hl=zh-cn)。


欲获取一个完整的包含了深层链接例子的导航项目，请参阅 [Navigation 代码实验室](https://codelabs.developers.google.com/codelabs/android-navigation/?hl=zh-cn#0)。








