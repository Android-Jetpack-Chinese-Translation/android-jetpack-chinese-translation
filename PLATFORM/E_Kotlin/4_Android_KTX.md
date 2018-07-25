# Android KTX
> 原文链接：[Android KTX  |  Android Developers](https://developer.android.google.cn/kotlin/ktx)  

Android KTX 是 Android Jetpack 里一个 Kotlin 的扩展库。它优化了 Kotlin 对 Jetpack 和 Android 原生 API 的调用，借助 Kotlin 语言中扩展函数/属性、lambda 表达式、命名参数、默认参数等一系列的功能，使 Kotlin 在 Android 的开发过程中变得更加简洁、愉快和自然。Android KTX 没有在已有的 Android API 的基础上增加任何新的功能。

你可以在 [DevBytes 的系列视频](https://www.youtube.com/watch?v=r_19VZ0xRO8&feature=youtu.be "DevBytes 的系列视频")中了解更多关于 Android KTX 的知识。

## 使用Android KTX
首先，在`build.gradle`中添加依赖：
```gradle
repositories {
    google()
}
```

Android KTX是模块化结构的。每个模块包含着一个或多个包。

要使用一个模块，首先在你的 `build.gradle` 文件中添加模块对应的 artifact。记得要在 artifact 后面加上版本号。比如，如果你想用 core-ktx 模块的话，完整的依赖应该是这个样子的：
```gradle
dependencies {
    implementation 'androidx.core:core-ktx:1.0.0-alpha1'
}
```
## 模块
Android KTX 是由以下这些 Maven artifact 组成的。点击包名，可以在扩展函数的概述中看到其包含的 API 的文档。

| 模块 (artifact)  | 版本  |  包 |
| ------------ | ------------ | ------------ |
| androidx.core:core-ktx  | 1.0.0-alpha1  |  详见下面的core模块包列表 |
| androidx.fragment:fragment-ktx  | 1.0.0-alpha1  |  [androidx.fragment.graphics](https://developer.android.com/reference/kotlin/androidx/fragment/app/package-summary#extension-functions-summary "androidx.fragment.graphics") |
|  androidx.palette:palette-ktx | 1.0.0-alpha1  | [androidx.palette.graphics](https://developer.android.com/reference/kotlin/androidx/palette/graphics/package-summary#extension-functions-summary "androidx.palette.graphics")  |
|androidx.sqlite:sqlite-ktx | 1.0.0-alpha1  |  [androidx.sqlite.db](https://developer.android.com/reference/kotlin/androidx/sqlite/db/package-summary#extension-functions-summary "androidx.sqlite.db") |
|  androidx.collection:collection-ktx |  1.0.0-alpha1 | [androidx.collection](https://developer.android.com/reference/kotlin/androidx/collection/package-summary#extension-functions-summary "androidx.collection")  |
|  androidx.lifecycle:lifecycle-viewmodel-ktx | 2.0.0-alpha1  |  [androidx.lifecycle](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary#extension-functions-summary "androidx.lifecycle") |
| androidx.lifecycle:lifecycle-reactivestreams-ktx  | 2.0.0-alpha1  | [androidx.lifecycle](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary#extension-functions-summary "androidx.lifecycle")  |
| android.arch.navigation:navigation-common-ktx  |1.0.0-alpha01|  [androidx.navigation](https://developer.android.com/reference/kotlin/androidx/navigation/package-summary#extension-functions-summary "androidx.navigation") |
|  android.arch.navigation:navigation-fragment-ktx | 1.0.0-alpha01  | [androidx.navigation.fragment](https://developer.android.com/reference/kotlin/androidx/navigation/fragment/package-summary#extension-functions-summary "androidx.navigation.fragment")   |
|android.arch.navigation:navigation-runtime-ktx   | 1.0.0-alpha01  |  [androidx.navigation](https://developer.android.com/reference/kotlin/androidx/navigation/package-summary#extension-functions-summary "androidx.navigation")  |
|android.arch.navigation:navigation-testing-ktx | 1.0.0-alpha01  |  [androidx.navigation.testing](https://developer.android.com/reference/kotlin/androidx/navigation/testing/package-summary#extension-functions-summary "androidx.navigation.testing")  |
| android.arch.navigation:navigation-ui-ktx  |  1.0.0-alpha01 | [androidx.navigation.ui](https://developer.android.com/reference/kotlin/androidx/navigation/ui/package-summary#extension-functions-summary "androidx.navigation.ui")   |
|android.arch.work:work-runtime-ktx |  1.0.0-alpha01 |[ androidx.work.ktx](https://developer.android.com/reference/kotlin/androidx/work/ktx/package-summary#extension-functions-summary " androidx.work.ktx")   |  |



Core 模块包含以下这些包：
- [androidx.core.animation](https://developer.android.com/reference/kotlin/androidx/core/animation/package-summary#extension-functions-summary "androidx.core.animation")
- [androidx.core.content](https://developer.android.com/reference/kotlin/androidx/core/content/package-summary#extension-functions-summary "androidx.core.content")
- [androidx.core.graphics](https://developer.android.com/reference/kotlin/androidx/core/graphics/package-summary#extension-functions-summary "androidx.core.graphics")
- [androidx.core.graphics.drawable](https://developer.android.com/reference/kotlin/androidx/core/graphics/drawable/package-summary#extension-functions-summary "androidx.core.graphics.drawable")
- [androidx.core.net](https://developer.android.com/reference/kotlin/androidx/core/net/package-summary#extension-functions-summary "androidx.core.net")
- [androidx.core.os](https://developer.android.com/reference/kotlin/androidx/core/os/package-summary#extension-functions-summary "androidx.core.os")
- [androidx.core.preference](https://developer.android.com/reference/kotlin/androidx/core/preference/package-summary#extension-functions-summary "androidx.core.preference")
- [androidx.core.text](https://developer.android.com/reference/kotlin/androidx/core/text/package-summary#extension-functions-summary "androidx.core.text")
- [androidx.core.transition](https://developer.android.com/reference/kotlin/androidx/core/transition/package-summary#extension-functions-summary "androidx.core.transition")
- [androidx.core.util](https://developer.android.com/reference/kotlin/androidx/core/util/package-summary#extension-functions-summary "androidx.core.util")
- [androidx.core.view](https://developer.android.com/reference/kotlin/androidx/core/view/package-summary#extension-functions-summary "androidx.core.view")
- [androidx.core.widget](https://developer.android.com/reference/kotlin/androidx/core/widget/package-summary#extension-functions-summary "androidx.core.widget")

## 示例
以下的示例展示了一些 Android KTX 的扩展函数。示例是按照模块（artifact）来分组的。如果想要查看完整的扩展函数列表，请参阅模块的文档。
#### androidx.core:core-ktx
Kotlin:
```kotlin
sharedPreferences.edit()
    .putBoolean("key", value)
    .apply()
```

Kotlin + Android KTX:
```kotlin
sharedPreferences.edit {
    putBoolean("key", value)
}
```

Kotlin:
```kotlin
view.viewTreeObserver.addOnPreDrawListener(
    object : ViewTreeObserver.OnPreDrawListener {
        override fun onPreDraw(): Boolean {
            viewTreeObserver.removeOnPreDrawListener(this)
            actionToBeTriggered()
            return true
        }
    }
)
```

Kotlin + Android KTX:
```kotlin
view.doOnPreDraw {
     actionToBeTriggered()
}
```
#### androidx.sqlite:sqlite-ktx
Kotlin:
```kotlin
db.beginTransaction()
try {
    // insert data
    db.setTransactionSuccessful()
} finally {
    db.endTransaction()
}
```

Kotlin + Android KTX:
```kotlin
db.transaction {
    // insert data
}
```

#### androidx.fragment:fragment-ktx
Kotlin:
```kotlin
supportFragmentManager
    .beginTransaction()
    .replace(R.id.my_fragment_container, myFragment, FRAGMENT_TAG)
    .commitAllowingStateLoss()
```
Kotlin + Android KTX:
```kotlin
supportFragmentManager.transaction(allowStateLoss = true) {
            replace(R.id.my_fragment_container, myFragment, FRAGMENT_TAG)
}
```

## 反馈
Android KTX 现在还处于 Alpha 阶段。我们会在 Jetpack 中持续添加新的 Kotlin 扩展函数。如果你想要报告问题或提出新功能的要求，请在 Android KTX 的[问题追踪表](https://issuetracker.google.com/issues/new?component=396204&template=1082185 "问题追踪表")里提交新的问题


