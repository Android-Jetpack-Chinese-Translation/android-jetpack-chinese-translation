# 切片模板
> 原文链接：[Slice templates  |  Android Developers](https://developer.android.google.cn/guide/slices/templates)

本节提供了如何使用 [Android Jetpack](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/A_Overview.md) 中的模板构造器来构建[切片]((https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_20_Slices/3_20_1_Overview.md))的详细信息。

## 定义您的切片模板

切片是通过 [`ListBuilder`](https://developer.android.google.cn/reference/androidx/slice/builders/ListBuilder?hl=zh-cn) 来构造的。`ListBuilder` 允许您为显示的列表中添加不同类型的行。本小节描述了每一种这样的行类型，以及它们是如何构造的。

### SliceAction
 
切片模板最基础的元素是 [`SliceAction`](https://developer.android.google.cn/reference/androidx/slice/builders/SliceAction?hl=zh-cn)，一个 `SliceAction` 含有一个有 [`PendingIntent`](https://developer.android.google.cn/reference/android/app/PendingIntent?hl=zh-cn) 的标签，并且是以下种类之一：

* 图标按钮
* 默认开关
* 自定义开关（一个有着开/关状态的 drawable）

`SliceAction` 用于本小节所描述的其他模板构造器中。它可以定义一个图像模式，用于决定该动作的图像是如何展示的：

* `ICON_IMAGE`：非常小的尺寸，可以染色（tint）
* `SMALL_IMAGE`：较小的尺寸，不可以染色
* `LARGE_IMAGE`：最大小的尺寸，不可以染色

### HeaderBuilder

在大多数情况下，您应当使用 [`HeaderBuilder`](https://developer.android.google.cn/reference/androidx/slice/builders/ListBuilder.HeaderBuilder?hl=zh-cn) 为您的模板设置一个列表的头部（header），用于支持下列内容：

* 标题
* 副标题
* 摘要副标题
* 主要操作

如下的是一些头部的配置范例。请注意，灰色盒子的位置代表的是潜在的图标和内边距：

![](https://developer.android.google.cn/guide/slices/images/headerbuilder-1.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/headerbuilder-2.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/headerbuilder-3.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/headerbuilder-4.png?hl=zh-cn)

#### 在不同界面绘制的头部

需要切片的时候，展示界面决定了切片应如何绘制。请注意，不同的宿主界面可能有不同的绘制方法。

小一些的版式一般只展示头部（如果头部存在的话）。如果您已经为头部指定了一个摘要，该摘要的文本就会取代副标题显示。

如果您并没在模板中指定一个头部，那么一般而言，第一个加入您 `ListBuilder` 的行会被显示。

![](https://developer.android.google.cn/guide/slices/images/headerbuilder-5.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/headerbuilder-6.png?hl=zh-cn)

#### HeaderBuilder 范例 - 有头部的简单列表

```kotlin
fun createSliceWithHeader(sliceUri: Uri) =
    list(context, sliceUri, ListBuilder.INFINITY) {
        setAccentColor(0xff0F9D) // Specify color for tinting icons
        header {
            title = "打个车"
            subtitle = "打车 4 分钟内到达"
            summary = "距离工作地点 1 消失 45 分钟 | 距离家 12 分钟"
        }
        row {
            title = "家"
            subtitle = "12 英里 | 12 分钟 | $9.00"
            addEndItem(
                IconCompat.createWithResource(context, R.drawable.ic_home),
                ListBuilder.ICON_IMAGE
            )
        }
    }
```

[`MySliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/MySliceProvider.kt#L66-L82)

> **注意**：简单起见，范例代码省略了上面图像中绘制的颜色。

#### 头部中的 SliceAction

切片列表头部还可以显示 SliceAction：

![](https://developer.android.google.cn/guide/slices/images/headeractions-1.png?hl=zh-cn)

```kotlin
fun createSliceWithActionInHeader(sliceUri: Uri): Slice {
    // 构造我们的切片操作
    val noteAction = SliceAction.create(
        takeNoteIntent,
        IconCompat.createWithResource(context, R.drawable.ic_pencil),
        ICON_IMAGE,
        "创建笔记"
    )

    val voiceNoteAction = SliceAction.create(
        voiceNoteIntent,
        IconCompat.createWithResource(context, R.drawable.ic_mic),
        ICON_IMAGE,
        "创建录音笔记"
    )

    val cameraNoteAction = SliceAction.create(
        cameraNoteIntent,
        IconCompat.createWithResource(context, R.drawable.ic_camera),
        ICON_IMAGE,
        "创建照片笔记"
    )

    // 构造列表
    return list(context, sliceUri, ListBuilder.INFINITY) {
        setAccentColor(0xfff4b4) // Specify color for tinting icons
        header {
            title = "创建新笔记"
            subtitle = "用这个笔记应用来轻松搞定"
        }
        addAction(noteAction)
        addAction(voiceNoteAction)
        addAction(cameraNoteAction)
    }
}
```

[`MySliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/MySliceProvider.kt#L86-L120)

### RowBuilder

您可以使用 [`RowBuilder`](https://developer.android.google.cn/reference/androidx/slice/builders/ListBuilder.RowBuilder?hl=zh-cn) 来构建一行内容。一个行可以支持下列的任何一种：

* 标题
* 副标题
* 起始项目：SliceAction、图标或时间戳
* 结束项目：SliceAction、图标或时间戳
* 主要操作

您可以下列方式合并不同行的内容，根据这些方式各自的限制条件：

* 起始项目不会在切片的第一行出现
* 结束项目不能是 `SliceAction` 对象和 `Icon` 对象的混杂
* 一行只能含有一个时间戳

如下图片展示了行内容的例子。请注意，灰色盒子的位置代表的是潜在的图标和内边距：

![](https://developer.android.google.cn/guide/slices/images/rowbuilder-1.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/rowbuilder-2.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/rowbuilder-4.png?hl=zh-cn)

#### RowBuilder 范例 - Wi-Fi 开关

如下的例子展示了一个有着主要操作和默认开关的行：

![](https://developer.android.google.cn/guide/slices/images/rowbuilder-5.png?hl=zh-cn)

```kotlin
fun createActionWithActionInRow(sliceUri: Uri): Slice {
    // 主要操作：打开 Wi-Fi 设置
    val wifiAction = SliceAction.create(
        wifiSettingsPendingIntent,
        IconCompat.createWithResource(context, R.drawable.ic_wifi),
        ICON_IMAGE,
        "Wi-Fi 设置"
    )

    // 开关操作 - 切换 Wi-Fi 的开关状态.
    val toggleAction = SliceAction.createToggle(
        wifiTogglePendingIntent,
        "Wi-Fi 开关",
        isConnected /* 已打开 */
    )

    // 构造父构造器
    return list(context, wifiUri, ListBuilder.INFINITY) {
        setAccentColor(0xff4285) // 指定图标/控件的染色
        row {
            title = "Wi-Fi"
            primaryAction = wifiAction
            addEndItem(toggleAction)
        }
    }
}
```

[`MySliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/MySliceProvider.kt#L124-L149)

### GridBuilder

您可以通过 [`GridBuilder`](https://developer.android.google.cn/reference/androidx/slice/builders/GridBuilder?hl=zh-cn) 来构造一个网格的内容。该网格支持以下的图像类型：

* `ICON_IMAGE`：非常小的尺寸，可以染色（tint）
* `SMALL_IMAGE`：较小的尺寸，不可以染色
* `LARGE_IMAGE`：最大小的尺寸，不可以染色

网格的单元格是通过 [`CellBuilder`](https://developer.android.google.cn/reference/androidx/slice/builders/GridBuilder.CellBuilder?hl=zh-cn) 构造的。一个单元格不能为空，但可以支持多达两行的文本和一个图像。

下列图片展示了网格的例子：

![](https://developer.android.google.cn/guide/slices/images/gridbuilder-1.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/gridbuilder-2.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/gridbuilder-3.png?hl=zh-cn)

#### GridBuilder 例子 - 附近餐馆

如下的例子展示了包含图像和文本的网格行。

![](https://developer.android.google.cn/guide/slices/images/gridbuilder-4.png?hl=zh-cn)

```kotlin
fun createSliceWithGridRow(sliceUri: Uri): Slice {
    // 构造父构造器
    return list(context, sliceUri, ListBuilder.INFINITY) {
        header {
            title = "有名的餐馆"
            primaryAction = SliceAction.create(
                pendingIntent, icon, ListBuilder.ICON_IMAGE, "有名的餐馆"
            )
        }
        gridRow {
            cell {
                addImage(image1, LARGE_IMAGE)
                addTitleText("顶级大厨")
                addText("0.3 英里")
                contentIntent = intent1
            }
            cell {
                addImage(image2, LARGE_IMAGE)
                addTitleText("家常小菜")
                addText("0.5 英里")
                contentIntent = intent2
            }
            cell {
                addImage(image3, LARGE_IMAGE)
                addTitleText("随意晚餐")
                addText("0.9 英里")
                contentIntent = intent3
            }
            cell {
                addImage(image4, LARGE_IMAGE)
                addTitleText("拉面小馆")
                addText("1.2 英里")
                contentIntent = intent4
            }
        }
    }
}
```

[`MySliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/MySliceProvider.kt#L153-L189)

### RangeBuilder

使用 [`RangeBuilder`](https://developer.android.google.cn/reference/androidx/slice/builders/ListBuilder.RangeBuilder?hl=zh-cn)，您可以构造包含一个进度条或者输入范围（例如滑块）的行。

下列图片展示了进度条和滑块的例子：

![](https://developer.android.google.cn/guide/slices/images/rangebuilder-1.png?hl=zh-cn)

![](https://developer.android.google.cn/guide/slices/images/rangebuilder-2.png?hl=zh-cn)

#### RangeBuilder 例子 - 滑块

下列例子展示了如何使用 `InputRangeBuilder` 来构建一个包含了音量滑块的切片。欲构建一个进度条的行，请使用 [`addRange()`](https://developer.android.google.cn/reference/androidx/slice/builders/ListBuilder.html?hl=zh-cn#addRange(androidx.slice.builders.ListBuilder.RangeBuilder))。

```kotlin
fun createSliceWithRange(sliceUri: Uri): Slice {
    return list(context, sliceUri, ListBuilder.INFINITY) {
        inputRange {
            title = "铃声音量"
            inputAction = volumeChangedPendingIntent
            max = 100
            value = 30
        }
    }
}
```

[`MySliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/MySliceProvider.kt#L193-L202)

## 延迟内容

您应当从 [`SliceProvider#onBindSlice`](https://developer.android.google.cn/reference/android/app/slice/SliceProvider.html?hl=zh-cn#onBindSlice(android.net.Uri,%20java.util.List%3Candroid.app.slice.SliceSpec%3E)) 中尽可能快地返回一个切片。耗费时间的调用可能导致显示问题，例如闪抖和突兀的调整尺寸。

如果您的切片内容并不能被快速加载，那么您应使用 `null` 或者占位符来构造您的切片，并在构造器中注明其内容还在加载中。一旦内容已经准备就绪，使用您切片的 URI 来调用 [`getContentResolver().notifyChange(sliceUri, null)`](https://developer.android.google.cn/reference/android/content/ContentResolver?hl=zh-cn#notifyChange(android.net.Uri,%20android.database.ContentObserver))。这将导致另一个对 `SliceProvide#onBindSlice` 的调用，并让您重新使用新的内容来构造切片。

### 延迟内容的范例 - 开车上班

在下面的开车上班的行中，和公司的距离是动态决定的，并且可能无法立即获取。范例代码展示了如何在内容还在加载的时候，使用 `null` 来作为占位符：

![](https://developer.android.google.cn/guide/slices/images/delayedcontent-1.png?hl=zh-cn)

```kotlin
fun createSliceShowingLoading(sliceUri: Uri): Slice {
    // 我们还在等待该时间的加载，
    // 因此通过重载的 setSubtitle 方法来设置副标题，
    // 并传入一个 true 旗语来在切片上标明这一点。
    return list(context, sliceUri, ListBuilder.INFINITY) {
        row {
            title = "开车上班"
            setSubtitle(null, true)
            addEndItem(IconCompat.createWithResource(context, R.drawable.ic_work), ICON_IMAGE)
        }
    }
}
```

[`MySliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/MySliceProvider.kt#L206-L216)

## 使用切片来处理禁用的滚动

展示您的切片的界面可能不支持在模板中的滚动功能。在这种情况下，您的部分内容可能就不会被显示。

下例展示了一个显示 Wi-Fi 网络列表的切片：

![](https://developer.android.google.cn/guide/slices/images/disabledscrolling-1.png?hl=zh-cn)

如果 Wi-Fi 列表过长且滚动功能被禁止，那么您可以添加一个 **查看更多（See more）** 的按钮，用以保证用户至少有一个可以看到列表全部项目的办法。您可以使用 [`addSeeMoreAction()`](https://developer.android.google.cn/reference/androidx/slice/builders/ListBuilder?hl=zh-cn#addseemoreaction) 来添加该按钮，如下所示：

```kotlin
fun seeMoreActionSlice(sliceUri: Uri) =
    list(context, sliceUri, ListBuilder.INFINITY) {
        // ...
        setSeeMoreAction(seeAllNetworksPendingIntent)
        // ...
    }
```

[`MySliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/MySliceProvider.kt#L220-L227)

显示效果如下所示：

![](https://developer.android.google.cn/guide/slices/images/disabledscrolling-2.png?hl=zh-cn)

点击 **查看更多** 就会发送 `seeAllNetworkPendingIntent`。

或者，如果您想提供一个自定义的信息或行，可以考虑添加一个 `RowBuilder`：

```kotlin
fun seeMoreRowSlice(sliceUri: Uri) =
    list(context, sliceUri, ListBuilder.INFINITY) {
        // ...
        seeMoreRow {
            title = "查看全部的可用网络"
            addEndItem(
                IconCompat.createWithResource(context, R.drawable.ic_right_caret), ICON_IMAGE
            )
            primaryAction = SliceAction.create(
                seeAllNetworksPendingIntent,
                IconCompat.createWithResource(context, R.drawable.ic_wifi),
                ListBuilder.ICON_IMAGE,
                "Wi-Fi 网络"
            )
        }
    }
```

[`MySliceProvider.kt`](https://github.com/android/snippets/blob/3985af9676187e4ebfb676569588ba92ea2ceb7c/slice/src/main/java/com/example/android/slice/kotlin/MySliceProvider.kt#L231-L247)

只有在以下条件至少满足一个的时候，通过上述方法添加的行或动作才会显示：

* 您切片的展示者已经禁用了视图的滚动功能
* 您的行内容无法在可用的空间内显示完全

## 合并模板

通过合并不同的模板行类型，您就能创建一个内容丰富的动态切片。例如，一个切片可以包含一个列表头部、一个包含单个图片的网格、和一个包含两个文本单元格的网格：

![](https://developer.android.google.cn/guide/slices/images/combinedtemplates-1.png?hl=zh-cn)

如下的切片含有一个列表头部行和一个包含三个单元格的网格：

![](https://developer.android.google.cn/guide/slices/images/combinedtemplates-2.png?hl=zh-cn)

