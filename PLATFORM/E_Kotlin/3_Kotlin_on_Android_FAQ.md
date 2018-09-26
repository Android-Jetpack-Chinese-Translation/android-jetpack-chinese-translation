# 关于在 Android 中使用 Kotlin 的常见问答
> 原文链接：[Kotlin on Android FAQ  |  Android Developers](https://developer.android.google.cn/kotlin/faq)


### 为什么 Android 将 Kotlin 语言选为了一等（first-class）开发语言？

Kotlin 是一门与 Android 兼容的，精简干练、表达力强的语言，而且在设计上就是类型和空值安全的。它能和 Java 无缝衔接，因此让爱好 Java 的开发者能在保持原有 Java 代码的同时，增量地添加 Kotlin 代码并利用 Kotlin 类库。此外，许多 Android 开发者已经发现 Kotlin 让编程变得更快更有趣，因此我们想更好地支持他们。您可以浏览更多关于 [Kotlin 和 Android](https://developer.android.google.cn/kotlin/) 的信息。
 
### 对于像我这样已经在使用 Kotlin 的人而言，有什么变化吗？

在短期内，我们认为您能注意到的最大的变化是：Android Studio 的 3.0 及更高版本已经包含了完善测试过的 Kotlin 支持。我们认为这将为您带来更加轻松而稳定的开发体验。

### 我该如何在 Android Studio 中使用 Kotlin 呢？
 
Android Studio 的 3.0 及更高版本完整支持 Kotlin。在之前的版本中，您需要添加 Kotlin 插件来使用 Kotlin，但现在所有新版本的 Android Studio 已经自带了这些工具。因此，毋须任何额外操作，您就能创建完全基于 Kotlin 的新项目，将 Java 代码转换成 Kotlin，调试 Kotlin 代码，等等。请参阅[开始动手在 Android 中使用 Kotlin](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/2_Get_Started_with_Kotlin.md)。

### 我该如何在 Android Studio 中调试 Kotlin 呢？
 
调试 Kotlin 代码和调试 Java 代码没什么两样。您没什么需要特别注意的。
 

### 针对 Kotlin 还有哪些额外的集成开发环境支持？例如：代码风格检查、自动补全、重构，等等。

以 Android Studio 3.0 为例，该集成开发环境已经包含了针对 Kotlin 的所有工具。然而，目前我们正在着力解决一些[已知的问题和限制](https://developer.android.google.cn/studio/preview/kotlin-issues.html)。

### Kotlin 的未来将会是怎样的？

我们欣然接纳 Kotlin 语言的原因之一，就是 JetBrains 对其深谋远虑的规划设计。Google 正在和 JetBrains 并肩打造出色的开发者体验：从语言，到框架，再到工具。并且，我们还将通力协作，将 Kotlin 迁移到非营利性组织中。

### Kotlin 是开源的吗？
 
我们倾向于按照 [Apache 许可证 2.0 版本](http://www.apache.org/licenses/LICENSE-2.0)来开源 Kotlin，而且大部分的 Kotlin 软件也获得了该协议的许可。然而，在整个项目努力符合该协议的同时，可能会有一些需要逐个特别处理的例外。例如，Kotlin 使用的某些第三方类库可能是按照不同的开源协议开源的，但仍能和 Apache 许可证 2.0 保持兼容。

### 我该如何在 Java 和 Kotlin 之间抉择？

您可以“我全都要”！只要您觉得合适，两个同时一起上也大丈夫萌大奶！想知道 Kotlin 是否适合自己的话，您可以[在 Android 上动手](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/2_Get_Started_with_Kotlin.md)，或者参阅这些 [Kotlin 学习资源](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/5_Resources_to_Learn_Kotlin.md)。

### 我可以在 Kotlin 中调用 Android 或其他 Java 的类库吗？

可以的。Kotlin 提供了与 Java 的互用性。这是一个允许 Kotlin 代码清晰透明地调用 Java 方法的设计，还配有能把仅支持 Kotlin 的功能暴露给 Java 代码的注释。没有使用任何 Kotlin 限定的语义的 Kotlin 文件都能无须任何注释地直接被 Java 代码引用。综上所示，这种设计允许你把 Java 代码和 Kotlin 代码在颗粒级别上混合。欲了解更多，请参阅 [Kotlin 的互用性](https://www.kotlincn.net/docs/reference/java-interop.html)文档。

### 你们为 Android API 提供 Kotlin 的语法参考吼不吼啊？

吼啊！我们正在努力为所有的 Android API 文档提供地道的 Kotlin 参考。尽管还没全部准备就绪，但您仍能在 [Kotlin 参考索引](https://developer.android.google.cn/reference/)中找到可用的 Kotlin 参考。如果您要找的是 Kotlin 核心库的参考，请参阅 [Kotlin 标准库](https://www.kotlincn.net/docs/reference/multiplatform.html)文档。

### 我能在同一个项目中同时使用 Java 和 Kotlin 源文件吗？

可以的。您想用多少 Kotlin 都行，利用 [Kotlin 与 Java 的互用性](https://www.kotlincn.net/docs/reference/java-interop.html)，您可以将其混入 Java 源码中。

### 我能和 Kotlin 一起使用 C++ 吗？
 
可以的。Kotlin 完整支持 JNI。您只需把 JNI 方法用 [`external` 修饰符](https://www.kotlincn.net/docs/reference/java-interop.html#在-kotlin-中使用-jni)标记即可。

### 我该如何向新项目中加入 Kotlin 呢？
 
您只需在 Android Studio 中创建一个新项目的时候勾选 **Include Kotlin support** 即可。欲了解更多细节，请参阅[开始动手在 Android 中使用 Kotlin](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/2_Get_Started_with_Kotlin.md)。
 
### 我该如何向现有项目中加入 Kotlin 呢？

在 **Project** 窗口中选择您的模块，再选择 **File** 菜单的 **New**，随便选择一个 Android 模板，之后将 **Source Language** 选为 **Kotlin**。欲了解更多细节，请参阅[开始动手在 Android 中使用 Kotlin](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/2_Get_Started_with_Kotlin.md)。

###  我该如何将 Java 代码转换成 Kotlin 呢？
 
打开一个 Java 源文件，选择 **Code** 菜单的 **Convert Java File to Kotlin File**。或者，创建一个新的 Kotlin 文件（**File** → **New** → **Kotlin File/Class**），再将您的 Java 代码拷贝粘贴到那个文件中：当出现对话框时，选择 **Yes** 来把代码转换成 Kotlin。欲了解更多细节，请参阅[开始动手在 Android 中使用 Kotlin](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/2_Get_Started_with_Kotlin.md)。

### 将来会有文档、范例、codelab 和模板的 Kotlin 版本吗？

我们正在努力让文档和教育材料尽可能地同时照顾到 Java 和 Kotlin 用户。同时，开发者可以充分利用 Kotlin 极为出色的与 Java 的互用性，并在 Android Studio 中将 Java 源码自动转换成 Kotlin。

### Kotlin 的[协程](https://www.kotlincn.net/docs/reference/coroutines.html)能在 Android 上用吗？还有 `async` 和 `await` 呢？
 
Kotlin 协程现在应该是能用的，但它们暂时还是一个实验性的功能设计。虽这么说，Kotlin 并不保证这个功能将来会怎么样，因此也不能保证它能在 Android 上工作。

### Kotlin 对于 APK 的大小和方法数有什么影响？

Kotlin 运行时会为您的 APK 添加大约 7000 个方法和 1MB 的空间。如果您用 Kotlin 来替代掉项目中的其他类库，比如 `Guava` 或 `RxJava`，那么净影响还能更小一些。如果您用 `Proguard` 针对 Kotlin 为发布的 APK 进行优化，就像针对其他代码和类库那样，新增的空间也能再减少一点。

### 使用 Kotlin 会影响性能吗？

Kotlin 并没有对性能的直接影响，但就像 Java 一样，您应当深思熟虑如何使用它。譬如，在集合实例之间反复拷贝内容会影响垃圾回收的性能，而调用一个接受非空类型的方法将会增加一个空值检查的方法调用（不过，您也可以通过 `-Xno-param-assertions` 来禁用编译器的运行时空值检查）。


### Kotlin 支持哪些版本的 Android 呢？

所有的版本！Kotlin 与 JDK 6 兼容，所以用 Kotlin 编写的应用可以安全地运行在老旧的 Android 版本上。

### 哪里还有更多关于使用 Kotlin 的信息？

请参阅 [Kotlin 学习资源](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/PLATFORM/E_Kotlin/5_Resources_to_Learn_Kotlin.md)。

