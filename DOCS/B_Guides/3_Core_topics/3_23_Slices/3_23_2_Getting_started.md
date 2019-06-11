# 开始动手
> 原文链接：[Getting Started  |  Android Developers](https://developer.android.google.cn/guide/slices/getting-started)

本节展示了如何在您的应用中设置环境并开发[切片](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_23_Slices/3_23_1_Overview.md)。

> **注意**：Android Studio 3.2 及更新版本包含了帮您开发切片的额外工具和功能。

* AndroidX 重构工具：如果您的项目使用了 AndroidX 类库，就必需使用该工具。
* 切片语法检查：该工具能捕捉到使用切片的常见的不合规用法。
* `SliceProdiver` 模板：使用 `SliceProdiver` 来处理样板代码。

## 下载并安装切片查看器

下载最新的样本[切片查看器 APK 发行版](https://github.com/googlesamples/android-SliceViewer/releases)，让您能在不实现 [`SliceView`](https://developer.android.google.cn/reference/androidx/slice/widget/SliceView?hl=zh-cn) API 情况下测试您的切片。您还可以克隆[切片查看器的源码](https://github.com/googlesamples/android-SliceViewer/releases/download/1.0.0-alpha1/slice-viewer.apk)。

> **注意**：切片查看器要求 Android 4.4（API 版本 19）或更新版本。

如果您的开发环境并没设置好 ADB，请参阅 [ADB 指南](https://developer.android.google.cn/studio/command-line/adb?hl=zh-cn) 获取更多信息。

在您下载 `slice-viewer.apk` 的目录中，运行下列命令来为您的设备安装切片查看器：

```bash
adb install -r -t slice-viewer.apk
```

## 运行切片查看器

您可以从 Android Studio 项目中或命令行来启动切片查看器：

### 从您的 Android Studio 项目中启动切片查看器

1. 在您的项目中，选择 **Run > Edit Configurations...**
2. 在左上角，单击绿色的加号键
3. 选择 **Android App**

	![](https://developer.android.google.cn/guide/slices/images/sliceviewer-setup-1.png?hl=zh-cn)
	
4. 在 **Name** 栏中输入 *slice*
5. 从 **Module** 下拉列表中选择您的应用模块
6. 在 **Launch Options** 中，从 **Launch** 下拉列表中选择 **URL**
7. 在 **URL** 栏中输入 `slice-<your slice URI>`。例如，`slice-content://com.example.your.sliceuri`

	![](https://developer.android.google.cn/guide/slices/images/sliceviewer-setup-2.png?hl=zh-cn)
	
8. 单击 **OK**

> **注意**：下一次您想要启动切片查看器来查看您的切片时，您就可以使用上面刚刚创建的配置。

### 通过 ADB（命令行）来启动切片查看器

从您的 Android Studio 中运行您的应用：

```bash
adb install -t -r <yourapp>.apk
```

运行下列命令来查看您的切片：

```bash
adb shell am start -a android.intent.action.VIEW -d slice-<your slice URI>
```

![](https://developer.android.google.cn/guide/slices/images/sliceviewer-1.png?hl=zh-cn)
*切片查看器展示了一个单个 WiFi 切片*

### 在同一个地方查看您所有的切片

除了启动单个切片外，您还可以在一个连续的列表中查看您所有的切片。

* 使用搜索栏来通过 URI （例如，`content://com.example.android.app/hello`）手动查找您切片。每次您查找一个切片，它都会被加入到列表中。
* 每次您使用切片 URI 来启动切片查看器的时候，该切片都会被加入到列表中。
* 您可以从列表中滑动删除一个切片。
* 单击切片的 URI 来查看一个只含有该切片的页面。这和使用切片 URI 来启动切片查看器的效果是一样的。

>**注意**：以列表展示时，切片的滚动是禁用的。请以单独模式启动切片查看器来测试您的滚动功能。

![](https://developer.android.google.cn/guide/slices/images/sliceviewer-2.png?hl=zh-cn)
*切片查看器展示了一个列表的切片*

### 使用不同的模式查看切片

展示切片的应用可以在运行时切换 [`SliceView#mode`](https://developer.android.google.cn/reference/androidx/slice/widget/SliceView.html?hl=zh-cn#MODE_LARGE)，因此您应当确保您的切片在各个模式下都如期表现。在页面的右上角选择菜单图标来切换模式。

![](https://developer.android.google.cn/guide/slices/images/sliceviewer-3.png?hl=zh-cn)
*模式被设置为“small”的切片查看器显示了单个切片*

## 构建您的第一个切片

欲构建一个切片，打开您的 Android Studio 项目，右键单击 `src` 包，然后选择 **New... > Other > Slice Provider**。这会创建一个 [`SliceProvider`](https://developer.android.google.cn/reference/androidx/slice/SliceProvider?hl=zh-cn) 的子类，将所需的 provider 条目添加到您的 `AndroidManifest.xml`，并为您的 `build.gradle` 文件添加所需的切片依赖关系。

针对 `AndroidManifest.xml` 做出的修改如下所示：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.app">
    ...
    <application>
        ...
        <provider android:name="MySliceProvider"
            android:authorities="com.example.android.app"
            android:exported="true" >
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.app.slice.category.SLICE" />
            </intent-filter>
        </provider>
        ...
    </application>

</manifest>
```

>**注意**：您可以安全地导出（export）`SliceProvider`，因为所有的权限检查都是在内部进行的。

如下的依赖关系被加入您的 `build.gradle`：

```gradle
dependencies {
// Java
    implementation "androidx.slice:slice-builders:(latest version)"
// Kotlin
    implementation "androidx.slice:slice-builders-ktx:(latest version)"
}
```

>**注意**：`SliceProvider` 模板在默认情况下是指向 AndroidX 类库的。如果您的项目使用了老旧的兼容库，请确保您修改了 `build.gradle` 文件并使其指向 `com.android.support:slices-builders:(latest version)`。

>**注意**：如果您使用的是 Kotlin，请注意 `slice-builders-ktx` 只在 AndroidX 中提供。如果您的项目使用了老旧的兼容库，请确保您修改了 `build.gradle` 文件并使其指向 `com.android.support:slices-builders:(latest version)`。

每个切片都含有一个绑定的 URI。当一个界面想要显示您的切片时，该界面就会向您的应用发送一个绑定请求。接着，您的应用处理该请求，并动态地通过 [`onBindSlice`](https://developer.android.google.cn/reference/androidx/slice/SliceProvider.html?hl=zh-cn#onBindSlice(android.net.Uri,%20java.util.List%3Candroid.app.slice.SliceSpec%3E)) 方法来动态地构建切片。之后，该界面就可以在合适的时机展示切片了。

如下的例子展示了一个检查 `/hello` URI 路径并返回一个 **Hello World** 切片的 `onBindSlice` 方法：

```kotlin
override fun onBindSlice(sliceUri: Uri): Slice? {
    val activityAction = createActivityAction()
    return if (sliceUri.path == "/hello") {
        list(context, sliceUri, ListBuilder.INFINITY) {
            row {
                primaryAction = activityAction
                title = "Hello World."
            }
        }
    } else {
        list(context, sliceUri, ListBuilder.INFINITY) {
            row {
                primaryAction = activityAction
                title = "URI 未识别."
            }
        }
    }
}
```
[`GettingStartedSliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/GettingStartedSliceProvider.kt#L35-L52)

利用您在上一小节创建的切片运行配置，将您 **Hello World** 切片的 URI（例如：`slice-content://com.android.example.slicesample/hello`）传入切片查看器：

![](https://developer.android.google.cn/guide/slices/images/slice-1-helloworld.png?hl=zh-cn)

## 可交互的切片

和通知类似，您可以把会被用户交互触发的 [`PendingIntent`](https://developer.android.google.cn/reference/android/app/PendingIntent?hl=zh-cn) 对象附加到切片上来处理点击事件。如下的例子启动了一个可以接收并处理这些 Intent 的 [`Activity`](https://developer.android.google.cn/reference/android/app/Activity?hl=zh-cn)。

```kotlin
fun createSlice(sliceUri: Uri): Slice {
    val activityAction = createActivityAction()
    return list(context, sliceUri, INFINITY) {
        row {
            title = "在应用中执行动作"
            primaryAction = activityAction
        }
    }
}

fun createActivityAction(): SliceAction {
    val intent = Intent(context, MainActivity::class.java)
    return SliceAction.create(
        PendingIntent.getActivity(context, 0, Intent(context, MainActivity::class.java), 0),
        IconCompat.createWithResource(context, R.drawable.ic_home),
        ListBuilder.ICON_IMAGE,
        "进入应用"
    )
}
```
[`GettingStartedSliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/GettingStartedSliceProvider.kt#L60-L78)

![](https://developer.android.google.cn/guide/slices/images/slice-2-toast.png?hl=zh-cn)

切片还支持其他在发送给应用的 Intent 中包含状态的输入类型，例如开关：

```kotlin
fun createBrightnessSlice(sliceUri: Uri): Slice {
    val toggleAction =
        SliceAction.createToggle(
            createToggleIntent(),
            "打开/关闭自适应亮度",
            true
        )
    return list(context, sliceUri, ListBuilder.INFINITY) {
        row {
            title = "自适应亮度"
            subtitle = "针对环境光来优化亮度"
            primaryAction = toggleAction
        }
        inputRange {
            inputAction = (brightnessPendingIntent)
            max = 100
            value = 45
        }
    }
}

fun createToggleIntent(): PendingIntent {
    val intent = Intent(context, MyBroadcastReceiver::class.java)
    return PendingIntent.getBroadcast(context, 0, intent, 0)
}
```
[`GettingStartedSliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/GettingStartedSliceProvider.kt#L82-L106)

接收者就可以检查它所收到的状态：

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        if (intent.hasExtra(Slice.EXTRA_TOGGLE_STATE)) {
            Toast.makeText(context, "开关：" + intent.getBooleanExtra(
                    Slice.EXTRA_TOGGLE_STATE, false),
                    Toast.LENGTH_LONG).show()
        }
    }

    companion object {
        const val EXTRA_MESSAGE = "信息"
    }
}
```

![](https://developer.android.google.cn/guide/slices/images/slice-3-switch.png?hl=zh-cn)

## 动态的切片

切片还可以包含动态的内容。如下的例子中，切片现在包含了它收到的广播数量：

```kotlin
fun createDynamicSlice(sliceUri: Uri): Slice {
    return when (sliceUri.path) {
        "/count" -> {
            val toastAndIncrementAction = SliceAction.create(
                createToastAndIncrementIntent("已点击。"),
                actionIcon,
                ListBuilder.ICON_IMAGE,
                "增加。"
            )
            list(context, sliceUri, ListBuilder.INFINITY) {
                row {
                    primaryAction = toastAndIncrementAction
                    title = "数量: ${MyBroadcastReceiver.receivedCount}"
                    subtitle = "点♂我！"
                }
            }
        }

        else -> {
            list(context, sliceUri, ListBuilder.INFINITY) {
                row {
                    primaryAction = createActivityAction()
                    title = "URI 未找到到。"
                }
            }
        }
    }
}

fun createToastAndIncrementIntent(s: String): PendingIntent {
    return PendingIntent.getBroadcast(
        context, 0,
        Intent(context, MyBroadcastReceiver::class.java)
            .putExtra(MyBroadcastReceiver.EXTRA_MESSAGE, s), 0
    )
}
```
[`GettingStartedSliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/GettingStartedSliceProvider.kt#L112-L147)

这个例子虽然显示了数量，但并不能自己更新。您可以修改广播接收器（broadcast receiver），让其使用 [`ContentResolver#notifyChange`](https://developer.android.google.cn/reference/android/content/ContentResolver?hl=zh-cn#notifychange) 来通知系统发生了变化。

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        if (intent.hasExtra(Slice.EXTRA_TOGGLE_STATE)) {
            Toast.makeText(
                context, "开关：" + intent.getBooleanExtra(
                    Slice.EXTRA_TOGGLE_STATE, false
                ),
                Toast.LENGTH_LONG
            ).show()
            receivedCount++;
            context.contentResolver.notifyChange(sliceUri, null)
        }
    }

    companion object {
        var receivedCount = 0
        val sliceUri = Uri.parse("content://com.android.example.slicesample/count")
        const val EXTRA_MESSAGE = "信息"
    }
}
```

![](https://developer.android.google.cn/guide/slices/images/slice-4-count.png?hl=zh-cn)

## 切片模板

切片支持一系列模板。欲了解更多有关模板选项和行为的信息，请参阅[切片模板](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_23_Slices/3_23_3_Slice_templates.md)。
