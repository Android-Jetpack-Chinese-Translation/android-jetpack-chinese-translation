# WorkManager 基础知识
> 原文链接：[WorkManager basics  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/workmanager/basics)

使用 WorkManager，您可以轻易设定一个任务并将其传递给系统来在您指定的情况下执行。

本小节的话题包括了最基础的 WorkManager 特性。您将学会如何设定一个任务、指定其执行的条件以及交由系统处理。您还将学会如何设定重复任务。

欲了解更多关于 WorkManager 的进阶知识，例如链式任务、传递并返回值等等，请参阅 [WorkManager 高级进阶](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_10_WorkManager/3_2_10_3_Advanced.md)。欲了解全部细节，请参阅 [WorkManager 参考文档](https://developer.android.google.cn/reference/androidx/work/package-summary)。

> **注意**：欲将 WorkManager 库导入您的 Android 项目，请参阅[为您的项目添加 Android 架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_2_Adding_Components_to_your_Project.md)。

## 类和概念

WorkManager 的 API 使用了几个不同的类。在一些情况下，您需要继承其中之一。

如下是最重要的 WorkManager 类：

* [**`Worker`**](https://developer.android.google.cn/reference/androidx/work/Worker.html)：指明您的任务需要做的具体工作。WorkManager 的 API 已经包含了一个 [`Worker`](https://developer.android.google.cn/reference/androidx/work/Worker.html) 的抽象类，您需要继承它来完成剩下的工作。
* [**`WorkRequest`**](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html)：代表一个具体的单个任务。一个 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 对象至少应指定工作由哪个 [`Worker`](https://developer.android.google.cn/reference/androidx/work/Worker.html) 来完成。然而，您也可以为 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 对象添加细节，例如任务应当在何种条件下执行。每个 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 都有一个自动生成的独特的 ID，您可以利用这个 ID 来取消一个加入队列的任务或者获取其状态。[`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 是一个抽象类，您需要在自己的代码里继承它的一个直属子类：[`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html) 或 [`PeriodicWorkRequest`](https://developer.android.google.cn/reference/androidx/work/PeriodicWorkRequest.html)。
	* [**`WorkRequest.Builder`**](https://developer.android.google.cn/reference/androidx/work/WorkRequest.Builder.html)：一个用于创建 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 对象的助手类。您仍需要使用其直属子类之一： [`OneTimeWorkRequest.Builder`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.Builder.html) 或 [`PeriodicWorkRequest.Builder`](https://developer.android.google.cn/reference/androidx/work/PeriodicWorkRequest.Builder.html)。
	* [**`Constraints`**](https://developer.android.google.cn/reference/androidx/work/Constraints.html)：指定任务运行时的限制条件，例如：“只在连接到网络时”。您可以使用 [`Constraints.Builder`](https://developer.android.google.cn/reference/androidx/work/Constraints.Builder.html) 来创建 [`Constraints`](https://developer.android.google.cn/reference/androidx/work/Constraints.html) 对象，并将 [`Constraints`](https://developer.android.google.cn/reference/androidx/work/Constraints.html) 在创建 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 之前传给 [`WorkRequest.Builder`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.Builder.html)。
* [**`WorkManager`**](https://developer.android.google.cn/reference/androidx/work/WorkManager.html)：将任务请求（work request）添加到队列中并管理它们。把 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 对象传给 [`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 来将其加入队列。[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 以这样的方式安排任务：在遵守您指定限制条件的前提下，均衡系统各资源的负载。
* [**`WorkInfo`**](https://developer.android.google.cn/reference/androidx/work/WorkInfo.html)：包含有关特定任务的信息。[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 为每个 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 提供了一个 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)，而后者持有一个 [`WorkInfo`](https://developer.android.google.cn/reference/androidx/work/WorkInfo.html) 对象；通过观察该 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)，您可以确定任务的当前状态，并在任务结束后获取任何返回值。

## 典型流程

假设您正在编写一个相册应用，而且该相册需要定期地压缩其存储的图像。您想要使用 WorkManager 的 API 来安排图像压缩的任务。在这个情境中，您不太在乎压缩发生的具体时间，而是只想设定这些任务后就撒手不管。

首先，您需要定义您的 [`Worker`](https://developer.android.google.cn/reference/androidx/work/Worker.html) 类，并覆写其 [`doWork()`](https://developer.android.google.cn/reference/androidx/work/Worker.html#doWork()) 方法。您的 Worker 类指定了执行操作的具体细节，但并不包含任何关于任务应当执行的信息。

```java
public class CompressWorker extends Worker {

    public CompressWorker(
        @NonNull Context context,
        @NonNull WorkerParameters params) {
        super(context, params);
    }

    @Override
    public Worker.Result doWork() {

        // 在这里完成工作——就这个例子而言是压缩存储的图像。
        // 本示例中并没有传递参数，因而该任务假设为“压缩整个图像库”。
        myCompress();

        // 根据您的返回值来标明任务是成功还是失败：
        return Result.success();

        // （返回 Result.retry() 会让 WorkManager 过会儿重试该任务；
        // 返回 Result.failure() 则是说明不需要重试。）
    }
}
```

接下来，您根据这个 [`Worker`](https://developer.android.google.cn/reference/androidx/work/Worker.html) 来创建一个 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html)，并使用 [`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 来将其加入队列：

```java
OneTimeWorkRequest compressionWork =
        new OneTimeWorkRequest.Builder(CompressWorker.class)
    .build();
WorkManager.getInstance().enqueue(compressionWork);
```

[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 会选择合适的时间来运行这个任务，平衡考虑各种因素，比如：系统负载、设备是否连接电源，等等。在大多数情况下，如果您不指定任何限制条件，[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 就会立即执行您的任务。如果您需要检查任务状态，只需通过相应的 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)<[`WorkInfo`](https://developer.android.google.cn/reference/androidx/work/WorkInfo.html)> 的句柄来获得其 [`WorkInfo`](https://developer.android.google.cn/reference/androidx/work/WorkInfo.html) 对象。例如，如果您想要检查一个任务是否已经完成，您就可以这样编写代码：

```java
WorkManager.getInstance().getWorkInfoByIdLiveData(compressionWork.getId())
    .observe(lifecycleOwner, workInfo -> {
        // 利用任务状态来做些事
        if (workInfo != null && workInfo.getState().isFinished()) {
            // ...
        }
    });
```

欲了解更多有关使用 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 的信息，请参阅 [LiveData 概览](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_5_LiveData.md)。

## 任务限制条件（Constraints）

只要您愿意，您就能指定任务运行时的限制条件。例如，您可能想要指定某个任务只在设备连接到电源并处于空闲状态时执行。在这种情况下，您需要创建一个 [`OneTimeWorkRequest.Builder`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.Builder.html) 对象，并使用该 builder 来创建真正的 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html) 对象。

```java
// 创建一个 Constraints 对象来定义任务运行时的限制条件
Constraints myConstraints = new Constraints.Builder()
    .setRequiresDeviceIdle(true)
    .setRequiresCharging(true)
    // 还有很多其他 constraints 可以用，
    // 请查看 Constraints.Builder 参考文档
     .build();

// ……然后创建一个 OneTimeWorkRequest 来使用上述限制条件
OneTimeWorkRequest compressionWork =
                new OneTimeWorkRequest.Builder(CompressWorker.class)
     .setConstraints(myConstraints)
     .build();
```

之后，仍旧是将新创建的 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html) 对象传入 [`WorkManager.enqueue()`](https://developer.android.google.cn/reference/androidx/work/WorkManager#enqueue(androidx.work.WorkRequest...))。[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 会在考虑到这些限制条件的情况下寻找时机来执行任务。

## 取消一个任务

您可以在把一个任务加入队列后将其取消。欲取消一个任务，您需要从 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 对象中获得其 ID。例如，如下的代码取消了上面提到的 `compressionWork`：

```java
UUID compressionWorkId = compressionWork.getId();
WorkManager.getInstance().cancelWorkById(compressionWorkId);
```

[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 会尽其所能地取消任务，但这从本质上来说还是不确定的——在您试图取消该任务时，它可能已经在执行或完成了。[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 还提供了取消一个 [唯一任务序列](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_10_WorkManager/3_2_10_3_Advanced.md) 当中、或有特定 [标签](https://developer.android.google.cn/topic/libraries/architecture/workmanager/basics#tag)（Tag）的全部任务的方法——当然，这仍只是尽力而为。

## 加注标签（Tag）的任务

您可以通过为任意 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 指定标签字符串的方式，来将您的任务按逻辑关系分组。欲设置一个标签，请按下例所示调用 [`WorkRequest.Builder.addTag()`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.Builder#addtag)：

```java
OneTimeWorkRequest cacheCleanupTask =
        new OneTimeWorkRequest.Builder(MyCacheCleanupWorker.class)
    .setConstraints(myConstraints)
    .addTag("清理")
    .build();
```

[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 提供了若干实用方法来让您操作特定标签下的所有任务。例如：[`WorkManager.cancelAllWorkByTag(String)`](https://developer.android.google.cn/reference/androidx/work/WorkManager#cancelAllWorkByTag(java.lang.String)) 能取消加注了特定标签的所有任务，而 [`WorkManager.getWorkInfosByTagLiveData(String)`](https://developer.android.google.cn/reference/androidx/work/WorkManager#getWorkInfosByTagLiveData(java.lang.String)) 能把加注了特定标签的所有任务的 [`WorkInfo`](https://developer.android.google.cn/reference/androidx/work/WorkInfo.html) 作为一个列表返回。

## 重复（Recurring）任务

您可能有个任务需要重复执行，例如：图片管理器应用可能不会只把其照片压缩仅仅一次——更可能的是，它会时不时检查自己的分享照片，来判断是否有新加入的或变更的图像需要被压缩。这种重复任务可以压缩新找到的图片，换言之，它能在新找到需要压缩的图片时，触发一个新的“压缩这张图片”的任务。

欲创建一个重复任务，请使用 [`PeriodicWorkRequest.Builder`](https://developer.android.google.cn/reference/androidx/work/PeriodicWorkRequest.Builder.html) 类来创建一个 [`PeriodicWorkRequest`](https://developer.android.google.cn/reference/androidx/work/PeriodicWorkRequest.html) 对象，并像 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html) 那样将其加入队列。例如，假设我们定义了一个 `PhotoCheckWorker` 类来辨别需要压缩的图像，如果您想要每隔12小时执行一次清点任务，您需要如下例所示地创建一个 [`PeriodicWorkRequest`](https://developer.android.google.cn/reference/androidx/work/PeriodicWorkRequest.html) 对象：

```java
PeriodicWorkRequest.Builder photoCheckBuilder =
        new PeriodicWorkRequest.Builder(PhotoCheckWorker.class, 12,
                                        TimeUnit.HOURS);
// ……您愿意的话，也可在此 builder 中设定限制条件……

// 创建实际的任务对象：
PeriodicWorkRequest photoCheckWork = photoCheckBuilder.build();
// 并将此重复任务入队
WorkManager.getInstance().enqueue(photoCheckWork);
```

[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 会尝试按您请求的时间间隔运行您的任务，具体取决于您施加的限制条件及其他要求。
