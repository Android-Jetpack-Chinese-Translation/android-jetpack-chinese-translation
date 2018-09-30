# 向您的项目中添加 Android 架构组件
> 原文链接：[Adding Components to your Project  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/adding-components)

在开始之前，我们建议您先阅读这篇[应用架构指南](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/B_Get_started/2_Guide_to_app_architecture.md)。该指南提及了一些对任何应用都有意义的原则，并且展示了应该如何把架构组件整合起来。

Android 架构组件可以从 Google 的 Maven 仓库中获取，步骤如下：

## 添加 Google Maven 仓库

Android Studio 的项目在默认状态下并没添加 Google Maven 仓库。

为了将其添加到您的项目中，打开**您的项目**（而非您的应用或模块）的 `build.gradle` 文件，然后添加 `google()` 仓库，如下所示：

```gradle
allprojects {
    repositories {
        jcenter()
        google()
    }
}
```

## 声明依赖关系

打开**您的应用或模块**的 `build.gradle` 文件，然后添加您需要依赖的 artifact。您可以添加下述的全部依赖，或者只添加其中的一部分。

### AndroidX

将来的架构组件将会被作为 AndroidX 的一部分来开发，而且将被编入每个稳定的依赖关系的列表中。为了符合 AndroidX 的标准，这些 alpha 版本的 artifact 将被重新命名。

欲了解有关 AndroidX 重构的更多细节、以及它会如何影响包和模块的 ID，请参阅 [AndroidX 重构](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/D_Libraries/6_AndroidX/6_2_Refactoring.md)。

### Kotlin

对于标记有 `// use -ktx for Kotlin` 的几个 AndroidX 依赖关系而言，只需进行如下所示的替换就能支持 Kotlin 扩展模块：

```gradle
implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version" // use -ktx for Kotlin
```

```gradle
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
```

欲了解包括 Kotlin 扩展的更多信息，请参阅 [Android KTX](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/4_Android_KTX.md)。

> **注意**：请务必在基于 Kotlin 的应用中使用 **kapt** 而非 **annotationProcessor**。此外，您还应该添加 **kotlin-kapt** 插件。

## 生命周期

包括了 [LiveData](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_5_LiveData.md) 和 [ViewModel](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_9_ViewModel.md) 在内的有关[处理生命周期](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_4_Handling_Lifecycles.md)的依赖。

```gradle
dependencies {
    def lifecycle_version = "1.1.1"

    // ViewModel 和 LiveData
    implementation "android.arch.lifecycle:extensions:$lifecycle_version"
    // 或者：只添加 ViewModel
    implementation "android.arch.lifecycle:viewmodel:$lifecycle_version" // use -ktx for Kotlin
    // 或者：只添加 LiveData
    implementation "android.arch.lifecycle:livedata:$lifecycle_version"
    // 或者：只添加 生命周期（既没有 ViewModel 也没有 LiveData）
    //     支持库需要导入这个轻量级的依赖
    implementation "android.arch.lifecycle:runtime:$lifecycle_version"

    annotationProcessor "android.arch.lifecycle:compiler:$lifecycle_version" // use kapt for Kotlin
    // 或者：如果您使用 Java8，请添加下面这个，而非上面的生命周期编译器
    implementation "android.arch.lifecycle:common-java8:$lifecycle_version"

    // 可选：对 LivaData 的 ReactiveStreams 支持
    implementation "android.arch.lifecycle:reactivestreams:$lifecycle_version"

    // 可选：对 LiveData 的测试辅助模块
    testImplementation "android.arch.core:core-testing:$lifecycle_version"
}
```

**AndroidX** 版本

```gradle
dependencies {
    def lifecycle_version = "2.0.0-beta01"

    // ViewModel 和 LiveData
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
    // 或者：只添加 ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version" // use -ktx for Kotlin
    // 或者：只添加 LiveData
    implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"
    // 或者：只添加 生命周期（既没有 ViewModel 也没有 LiveData）
    //     某些 AndroidX 的 UI 库需要导入这个轻量级的依赖
    implementation "androidx.lifecycle:lifecycle-runtime:$lifecycle_version"

    annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version" // use kapt for Kotlin
    // 或者：如果您使用 Java8，请添加下面这个，而非上面的生命周期编译器
    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

    // 可选：对 LivaData 的 ReactiveStreams 支持
    implementation "androidx.lifecycle:lifecycle-reactivestreams:$lifecycle_version" // use -ktx for Kotlin

    // 可选：对 LiveData 的测试辅助模块
    testImplementation "androidx.arch.core:core-testing:$lifecycle_version"
}
```

## Room

包括了 [测试 Room 迁移](https://developer.android.google.cn/topic/libraries/architecture/room.html#db-migration-testing) 和 [Room RxJava](https://developer.android.google.cn/training/data-storage/room/accessing-data#query-rxjava) 在内的有关 [Room 数据持久化库](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md)的依赖。

```gradle
dependencies {
    def room_version = "1.1.1"

    implementation "android.arch.persistence.room:runtime:$room_version"
    annotationProcessor "android.arch.persistence.room:compiler:$room_version" // use kapt for Kotlin

    // 可选：对 Room 的 RxJava 支持
    implementation "android.arch.persistence.room:rxjava2:$room_version"

    // 可选：对 Room 的 Guava 支持，包括了 Optional 和 ListenableFuture
    implementation "android.arch.persistence.room:guava:$room_version"

    // 测试辅助模块
    testImplementation "android.arch.persistence.room:testing:$room_version"
}
```

**AndroidX** 版本

```gradle
dependencies {
    def room_version = "2.0.0-beta01"

    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version" // use kapt for Kotlin

    // 可选：对 Room 的 RxJava 支持
    implementation "androidx.room:room-rxjava2:$room_version"

    // 可选：对 Room 的 Guava 支持，包括了 Optional 和 ListenableFuture
    implementation "androidx.room:room-guava:$room_version"

    // 测试辅助模块
    testImplementation "androidx.room:room-testing:$room_version"
}
```

## Paging

有关 [Paging](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_1_Overview.md) 的依赖。

```gradle
dependencies {
    def paging_version = "1.0.0"

    implementation "android.arch.paging:runtime:$paging_version"

    // 或者：不添加测试用的 Android 依赖
    testImplementation "android.arch.paging:common:$paging_version"

    // 可选：RxJava 支持（目前处于最终测试版本）
    implementation "android.arch.paging:rxjava2:1.0.0-rc1"
}
```

**AndroidX** 版本

```gradle
dependencies {
    def paging_version = "2.0.0-beta01"

    implementation "androidx.paging:paging-runtime:$paging_version"

    // 或者：不添加测试用的 Android 依赖
    testImplementation "androidx.paging:paging-common:$paging_version"

    // 可选：RxJava 支持（目前处于最终测试版本）
    implementation "androidx.paging:paging-rxjava2:$paging_version"
}
```

## Navigation

有关 [Navigation](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_6_Navigation/3_2_6_3_Implement_Navigation_with_the_Navigation_Architecture_Component.md) 的依赖。

```gradle
dependencies {
    def nav_version = "1.0.0-alpha06"

    implementation "android.arch.navigation:navigation-fragment:$nav_version" // use -ktx for Kotlin
    implementation "android.arch.navigation:navigation-ui:$nav_version" // use -ktx for Kotlin

    // 可选：测试辅助模块
    androidTestImplementation "android.arch.navigation:navigation-testing:$nav_version" // use -ktx for Kotlin
}
```

`androidx.navigation` 包中的 Navigation 类已能够使用，但目前依赖于 27.1.1 版本的支持库，并且和架构组件有关联。依赖于 AndroidX 的 Navigation 版本将在未来不久放出。

## 安全参数

对于[安全参数](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_6_Navigation/3_2_6_3_Implement_Navigation_with_the_Navigation_Architecture_Component.md)，请将如下的 **classpath** 添加到您项目**最顶层**的 `build.gradle` 文件，

```gradle
buildscript {
    repositories {
        google()
    }
    dependencies {
        classpath "android.arch.navigation:navigation-safe-args-gradle-plugin:1.0.0-alpha06"
    }
}
```

和您的应用或模块的 `build.gradle` 文件。

```gradle
apply plugin: "androidx.navigation.safeargs"
```

## WorkManager

有关 [WorkManager](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_10_WorkManager/3_2_10_1_Overview.md) 的依赖。

```gradle
dependencies {
    def work_version = "1.0.0-alpha09"

    implementation "android.arch.work:work-runtime:$work_version" // use -ktx for Kotlin

    // 可选：Firebase JobDispatcher 支持
    implementation "android.arch.work:work-firebase:$work_version"

    // 可选：测试辅助模块
    androidTestImplementation "android.arch.work:work-testing:$work_version"
}
```

`androidx.work` 包中的 WorkManager 类已能够使用，但目前依赖于 27.1.1 版本的支持库，并且和架构组件有关联。依赖于 AndroidX 的 WorkManager 版本将在未来不久放出。

WorkManager 需要 `compileSdk` 高于版本 28。


欲了解更多信息，请参阅[添加构建依赖关系](https://developer.android.google.cn/studio/build/dependencies)。



