# 保存 UI 状态
> 原文链接：[Saving UI States  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/saving-states)

在应用被系统发起或销毁时，及时地保存并恢复 activity 的 UI 状态，对用户体验而言至关重要。在这些情况下，用户期待 UI 的状态能保持一直，但系统总会销毁 activity 及其所有的 UI 状态。

为了填补用户期待和系统行为之间的差距，请结合使用 [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel.html?hl=zh-cn) 对象、[`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onsaveinstancestate) 方法和本地存储的策略，来在上述应用和 activity 实例切换时持久化 UI 状态。决定具体使用哪些策略，须取决于您 UI 数据的复杂度、应用的用例，以及在恢复速度和内存消耗之间的权衡。

无论您使用哪种方法，您都应当保证应用满足用户对其 UI 状态的期待，并提供一个平滑轻快的 UI，以避免加载数据导致的延迟卡顿，尤其是频繁发生的配置变更（例如：屏幕旋转）。

本节讨论了用户对于 UI 状态的期待、保存状态的可选策略，以及各自的权衡和限制。

## 用户期待和系统行为

在用户期待 activity 的状态被清除或者保存时，他们会做出不同的行为。在一些情况下，系统能自动做出用户所期待的行为；而在另一些情况下，系统会做出恰恰相反的行为。

### 用户发起的 UI 状态解除（dismiss）

用户期待的是：当他们启动一个 activity 的时候，其 UI 的临时状态会一直保持相同，直到用户彻底解除该 activity。

用户可以通过下列途径彻底解除一个 activity：

* 点击返回按钮
* 从“概览（最近应用）”屏幕中划掉应用
* 从该 activity 导航到上级页面
* 从“设置”页面强制停止应用
* 完成应用的某种由 `activity.finish()` 支持的“完成”操作

在这些途径中，用户的设想是他们已经永久地导航离开了 activity，并且他们如果再次打开应用就应当看到一个干净的状态。系统针对这些解除场景的底层行为是能够满足用户期待的：activity 的实例、其存储的任何状态和保存的实例状态都会被销毁并从内存中移除。

这种完全解除的规则有一些特例：例如，某个用户可能期待浏览器重新打开他上次使用后退键退出应用时浏览的页面。

### 系统发起的 UI 状态解除

用户期待的是：一个 activity 的 UI 状态在配置变更（例如，屏幕旋转、切换到多窗口模式）前后保持一致。然而，系统在这种配置变更发生时的默认行为是销毁 activity，抹除其实例中储存的所有 UI 状态。欲了解更多有关设备配置的信息，请参阅[配置参考页面](https://developer.android.com/reference/android/content/res/Configuration.html?hl=zh-cn#lfields)。请注意，配置变更的默认行为是可以被覆写的，尽管并不被推荐。欲了解更多细节，请参阅[自行处理配置变更](https://developer.android.com/guide/topics/resources/runtime-changes.html?hl=zh-cn#HandlingTheChange)。

用户在临时切换到另一个应用并在一段时间后回到您的应用时，也会期待您 activity 的 UI 状态能够保持一致。例如，用户在您的搜索 activity 中执行了一个查询，并点击主页键或者接了一个电话，那么当他期待的是回到搜索 activity 时仍能看到和当时完全一样的查询关键词以及搜索结果。

在这种情景下，您的应用被放到后台运行，系统会尽力在内存中保持您的应用进程。然而，在用户和其他应用交互时，系统可能销毁您的应用进程。那样的话，activity 实例及其储存的状态就被销毁了。当用户重新启动该应用时，activity 会意外地显示空白的状态。欲了解更多有关进程死亡的信息，请参阅[进程和应用的生命周期](https://developer.android.com/guide/components/activities/process-lifecycle.html?hl=zh-cn)。

## 保存应用状态的可选策略

当系统的默认行为并不契合用户对 UI 状态的期待时，您必须存储并恢复用户的 UI 状态，以确保用户不会察觉到系统发起的销毁。

每种保存 UI 状态的可选策略都会随着下列影响用户体验的指标而变化：

| | ViewModel | 保存的实例状态 | 持久化存储 |
|---|---|---|---|
| 存储位置 | 内存中 | 序列化到硬盘上 | 在硬盘上或云端 |
| 从配置变更中存活 | 能 | 能 | 能 |
| 从系统发起的进程销毁中存活 | 不能 | 能 | 能 |
| 从用户发起的 activity 解除 / `onFinish()` | 不能 | 不能 | 能 |
| 数据限制 | 支持复杂的对象，但空间受可用内存的影响 | 只适合原始类型和 String 等简单的小对象 | 只受硬盘空间或从云端获取的成本/时间的影响 |
| 读/写时间 | 快（只从内存读取）| 慢（要求序列化/反序列化和硬盘读取）| 慢（要求磁盘读取或网络通讯）|

## 使用 ViewModel 来处理配置变更

在用户活跃使用应用时，ViewModel 是存储和管理 UI 相关数据的理想手段。它支持对 UI 数据的快速访问，并帮助您避免在屏幕旋转、窗口大小调整和其他常见的配置变更时重新从网络或硬盘读取数据。欲了解如何实现一个 ViewModel，请参阅 [ViewModel 概览](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_9_ViewModel.md)。

ViewModel在内存中保持数据，因而比从硬盘或网络获取数据要快得多。一个 ViewModel 是和一个 activity 或其他生命周期所有者绑定的，所以在配置变更时能保持在内存中、并由系统自动将其绑定到新的 activity 实例上。

当用户退出您的 activity、fragment 或您的代码调用 finish() 时，系统会自动销毁 ViewModel，因此状态就会如用户期待的那样被清除。

和保存的实例状态不同，ViewModel 是在系统发起的进程死亡中被销毁的。这就是为什么您应当配合 `onSaveInstanceState()` 一起使用 VideModel 对象的原因：把识别符贮藏到保存的实例状态中有助于 ViewModel 在进程死亡后重新加载数据。

如果你已经有了一个在内存中储存您的 UI 状态的配置变更解决方案，那么您就可能无须使用 ViewModel 了。

## 使用 onSaveInstanceState() 作为处理系统发起的进程死亡的备份

如果系统销毁并不久之后重建 UI 控制器（如 activity 或 fragment），[`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onsaveinstancestate) 回调方法能储存用于重新加载这些 UI 控制器状态的数据。欲了解如何实现保存的实例状态，请参阅 [Activity 生命周期指南](https://developer.android.com/guide/components/activities/activity-lifecycle.html?hl=zh-cn#asem) 中的“保存和恢复 activity 状态。

保存的实例状态的 bundle 能存活过配置变更和进程死亡，但仍受存储空间和速度的限制，因为该方法需要把数据序列化到硬盘上。如果数据很复杂的话，序列化就会消耗大量内存。由于配置变更时该过程是在主线程上进行的，耗时过久的序列化可能导致掉帧和界面卡顿。

请不要使用 onSaveInstanceState 来储存大量数据（如位图）或需要长时间序列化/反序列化的复杂数据结构；相反地，请只储存原始类型和简单的小对象（例如 String）。这样的话，即使其他的持久化机制都失败了，只储存少量的必要数据（例如 ID）的 onSaveInstanceState() 也能重建必要的数据，用于将 UI 恢复到之前的状态。

根据您应用的用例，您可能不想总是使用 [`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onsaveinstancestate)。例如，浏览器可能自动恢复用户上次退出浏览器时正在浏览的页面。如果您的 activity 需要这样的行为，您就可以放弃使用 [`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onsaveinstancestate)，而选择把所有数据都持久化到本地。

此外，如果您从 intent 中打开一个 activity，那么 附加信息（extras）的 bundle 就会在配置变更发生时、以及系统创建 activity 时被两次传入 activity。如果某个 UI 状态数据（例如搜索的关键词）是从该 intent 的附加信息中传递的，那么您需要使用这些附加信息的、而非 onSaveInstanceState() 的 bundle。欲了解更多有关 intent 的附加信息，请参阅 [Intent 和 Intent 过滤器](https://developer.android.com/guide/components/intents-filters.html?hl=zh-cn)。

无论是以上哪种场景，您仍应使用 [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel.html?hl=zh-cn) 来在配置变更发生时避免浪费从数据库中重新加载的时间。

在保存 UI 数据很简单轻量的情况下，您可以只使用 [`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onsaveinstancestate) 来保存您的状态数据。

## 使用本地持久化存储来处理进程死亡时的大量复杂数据

只要您的应用是安装在用户的设备上的，数据库或 Shared Preferences 等本地持久化存储就能一直有效存在，除非用户手动清除了您的应用数据。然而，尽管这些本地存储能够存活过系统发起的 activity 和应用进程死亡，用它们来获取数据的成本也是很高的，因为它们需要把数据从硬盘读取到内存中。一般而言，这种本地持久化存储可能已经是您应用架构的一部分，用于存储所有您在打开/关闭 activity 时不想丢失的数据了。

无论是 ViewModel 还是 保存的实例状态，都不能用作长期的存储解决方案，因而不能替代数据库等本地存储。您应当使用这些机制来暂时储存易丢失的 UI 状态，并使用持久化存储来储存其他应用数据。欲了解更多利用本地存储来长期（例如，从设备重启中存活）持久化您的数据模型的细节，请参阅[架构应用指南](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/B_Get_started/2_Guide_to_app_architecture.md)。

## 管理 UI 状态：分而治之

通过把任务分别交给不同的持久化机制来处理，您可以高效地存储和恢复应用状态。在大多数情况下，这每一种机制都应根据数据的复杂度、访问速度和生命周期，来各自储存一种特定类型的用于 activity 中的数据：

* 本地持久化：储存所有您在打开/关闭 activity 时不愿丢失的数据。
	* 例如：一组歌曲的集合，包含音频文件和元数据。
*  [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel.html?hl=zh-cn)：在内存中储存所有用于展示相关 UI 控制器的数据。
	*  例如：最近搜索关键词，以及搜索结果中的歌曲对象。
*  [`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity.html?hl=zh-cn#onSaveInstanceState(android.os.Bundle))：储存少量用于在系统停止并重建 UI 控制器时、简单地重新加载 activity 状态的数据。相对于在这种情况下储存复杂的对象，该机制将复杂的对象持久化到本地存储，并在   [`onSaveInstanceState()`](https://developer.android.com/reference/android/app/Activity.html?hl=zh-cn#onSaveInstanceState(android.os.Bundle)) 的调用中为其储存独一无二的识别符。
	*  例如：储存最近的搜索关键词

譬如，考虑一个允许您在歌曲库中进行搜索的 activity 的例子。其不同的事件应分别如下处理：

1. 当用户添加一首歌曲时，`ViewModel` 立即把该数据的本地持久化委托下去。如果该歌曲应当在 UI 中展示，那么您还应更新 `ViewModel` 对象中的数据，用于反应该歌曲的新增。请务必确保所有的数据库插入操作都是在非主线程中执行的。
2. 当用户搜索一首歌曲时，无论您将从数据库中加载多么复杂的歌曲数据到 UI 控制器，您都应立即将其储存到 `ViewModel` 对象中。您还应将查询内容储存到 `ViewModel` 中。
3. 当 activity 被放入后台时，系统会调用 `onSaveInstanceState()`。您应当把查询内容储存到 `onSaveInstanceState()` 的 bundle 中。这样的少量数据是容易储存的，更包含了把 activity 恢复到之前状态所需的全部数据。

## 恢复复杂的状态：破镜重圆、完璧归赵

当用户回到 activity 时，有两种重建 activity 的可能场景：

* activity 是在被系统停止后被重建的：activity 已经在 `onSaveInstanceState()` 的 bundle 中储存了查询内容，并应该已经把查询内容传给了 `ViewModel`。`ViewModel` 发现它没有缓存任何搜索结果，因此把加载该查询内容的搜索结果的工作委托下去。
* activity 是在配置变更后被重建的：activity 已经在 `onSaveInstanceState()` 的 bundle 中储存了查询内容，并且 `ViewModel` 已经把搜索结果缓存好了。您应当把 `onSaveInstanceState()` 的 bundle 中的查询内容传给 `ViewModel`，并让其决定是否已经加载了所需的数据以及是否需要重新查询数据库。

> **注意**：当一个 activity 刚被创建时，`onSaveInstanceState()` 的 bundle 并不包含任何数据，并且 `ViewModel` 对象也是空的。当您创建 `ViewModel` 对象时，您传入的是一个空的搜索内容，用于告诉 `ViewModel` 对象其实并没有加载任何数据。因此，activity 是在一个空白状态中开始的。





