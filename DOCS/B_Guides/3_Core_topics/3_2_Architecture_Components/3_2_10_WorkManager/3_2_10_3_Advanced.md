# WorkManager 高级进阶
> 原文链接：[Advanced WorkManager topics  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/workmanager/advanced)

WorkManager 简化了精密任务请求的设定和安排，您可以在如下情境中使用其 API：

* 任务链式请求（Chained sequence）：按照一个指定顺序执行的任务。
* 唯一命名的序列（Unique named sequence）：在应用同时执行两个序列时做决断。
* 传递并返回值的任务，包括链式任务中每个任务都将参数传给下一个的情况。

## 链式任务（Chained task）

您的应用可能想要按某个特定的顺序执行一系列任务。[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 允许您创建一个任务序列并将其加入队列，该序列指定了多个任务及其执行的顺序。

例如，假设您的应用有三个 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html): `workA`、`workB` 和 `workC`，且它们必须按这个顺序执行。欲将其加入队列，将第一个任务传入 [`WorkManager.beginWith()`](https://developer.android.google.cn/reference/androidx/work/WorkManager#beginWith(androidx.work.OneTimeWorkRequest...)) 来创建一个序列，该方法返回一个定义了任务序列的 [`WorkContinuation`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation.html) 对象；接着，使用 [`WorkContinuation.then()`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation#then) 来将剩余的 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html) 按顺序加入序列；最后，使用 [`WorkContinuation`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation#enqueue) 来将整个序列加入队列。

```java
WorkManager.getInstance()
    .beginWith(workA)
        // 注意：WorkManager.beginWith() 返回一个
        // WorkContinuation 对象，因而下面调用的都是
        // WorkContinuation 的方法
    .then(workB)    // 怕您忘了，then() 返回的是一个新的 WorkContinuation 实例
    .then(workC)
    .enqueue();
```

[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 会根据每个任务的限制条件，按照请求的顺序执行它们。如果任意一个返回了 [`Worker.Result.FAILURE`](https://developer.android.google.cn/reference/androidx/work/Worker.Result#FAILURE)，整个序列就会结束。

您也可以向任意 [`.beginWith()`](https://developer.android.google.cn/reference/androidx/work/WorkManager#beginWith(androidx.work.OneTimeWorkRequest...)) 或 [`.then()`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation#then) 中传入多个 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html) 对象。那样的话，[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 会在执行剩余序列前将它们全部（并行地）执行。例如：

```java
WorkManager.getInstance()
    // 首先，（并行地）执行所有 A 任务：
    .beginWith(workA1, workA2, workA3)
    // ...当所有 A 任务都执行结束后，执行一个 B 任务：
    .then(workB)
    // ...之后，再（按任意顺序）执行 C 任务：
    // （译者注：这是因为 C 任务之后没有其他任务了）
    .then(workC1, workC2)
    .enqueue();
```

您还可以通过 [`WorkContinuation.combine()`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation#combine) 方法来并合多个任务链，从而创建更加复杂的序列。例如，假设您想要执行一个如下图所示的序列：

![**图一** 您可以用 WorkContinuation 来设置复杂的链式任务](https://developer.android.google.cn/images/topic/libraries/architecture/workmanager-chain.svg)

欲设置这样的序列，分别创建两个任务链，再将其并合到第三个中：

```java
WorkContinuation chain1 = WorkManager.getInstance()
    .beginWith(workA)
    .then(workB);
WorkContinuation chain2 = WorkManager.getInstance()
    .beginWith(workC)
    .then(workD);
WorkContinuation chain3 = WorkContinuation
    .combine(chain1, chain2)
    .then(workE);
chain3.enqueue();
```

在这个例子中，[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 在 `workB` 前执行 `workA`，同时在 `workD` 前执行 `workC`。只有 `workB` 和 `workD` 都完成后，[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 才执行 `workE`。

> **注意**：当 [`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 按顺序执行每个子链（subchain）时，它并不保证 **chain1** 中的任务是否会和 **chain2** 中的重叠。例如，**workB** 可能会在 **workC** 之前或之后执行，甚至可能同时执行。唯一可以确定的是，每个子链里的任务是按顺序执行的，也即：**workB** 在 **workA** 结束之前决不会开始。

[`WorkContinuation`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation#enqueue) 的方法还有许多变体，用于简化处理特定的情况。例如，方法 [`WorkContinuation.combine(OneTimeWorkRequest, WorkContinuation…)`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation#combine(androidx.work.OneTimeWorkRequest,%20androidx.work.WorkContinuation...)) 指示 [`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 在完成所有指定的 [`WorkContinuation`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation#enqueue) 之后，再以执行 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html) 作为结束。欲了解更多细节，请参阅 [`WorkContinuation`](https://developer.android.google.cn/reference/androidx/work/WorkContinuation.html) 参考文档。

## 唯一任务序列（Unique work sequence）

您可以通过调用 [`beginUniqueWork()`](https://developer.android.google.cn/reference/androidx/work/WorkManager#beginUniqueWork(java.lang.String,%20androidx.work.ExistingWorkPolicy,%20androidx.work.OneTimeWorkRequest...)) 而非 [`beginWith()`](https://developer.android.google.cn/reference/androidx/work/WorkManager#beginWith(androidx.work.OneTimeWorkRequest...)) 来创建一个唯一的任务序列。每个唯一任务序列都有一个名字，[`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 只允许一个任务序列同时使用相同的名字。当您创建一个新的唯一任务序列时，您可以指定 [`WorkManager`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html) 在已存在一个尚未完成的名字相同的任务序列时采取如下措施：
* 取消已存在的序列，并用新的将其[替换（replace）](https://developer.android.google.cn/reference/androidx/work/ExistingWorkPolicy#replace)。
* [保持（keep）](https://developer.android.google.cn/reference/androidx/work/ExistingWorkPolicy#keep) 已存在的序列，并忽略您的新请求。
* [追加（append）](https://developer.android.google.cn/reference/androidx/work/ExistingWorkPolicy#append) 您的新序列到已存在的序列，并在已存在序列的最后一个任务结束后执行新序列的第一个任务。

当您的任务不应被多次加入队列时，唯一任务序列就能起到很大的作用。例如，如果您的应用需要将其数据与网络同步，您就能把一个命名为“同步”的任务序列加入队列，并指定 WorkManager 在已经存在一个相同名字的任务序列时将新的请求忽略。
唯一任务序列在您需要逐渐构建一个任务长链时也很有用。例如，一个图片编辑应用可能允许用户取消一个很长的链式操作，其中的每个取消操作都可能花费一段时间执行，但它们必须按照正确的顺序执行。在这种情况下，应用可以创建一个命名为“取消”的任务链，并按需将每个取消操作追加到任务链末尾。

## 输入参数并返回值

为了获得更好的灵活性，您可以将参数传入您的任务并让其返回结果。传入和返回的值都是键值对。
欲将一个参数传入至一个任务中，请在创建 [`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.html) 之前调用 [`WorkRequest.Builder.setInputData()`](https://developer.android.google.cn/reference/androidx/work/WorkRequest.Builder#setinputdata) 方法。该方法接收一个使用 [`Data.Builder`](https://developer.android.google.cn/reference/androidx/work/Data.Builder.html) 创建的 [`Data`](https://developer.android.google.cn/reference/androidx/work/Data.html) 对象。[`Worker`](https://developer.android.google.cn/reference/androidx/work/Worker.html) 类可以通过调用 [`Worker.getInputData()`](https://developer.android.google.cn/reference/androidx/work/Worker#getinputdata) 来获取这些参数。
欲输出一个返回值，请在任务中调用 [`Worker.setOutputData()`](https://developer.android.google.cn/reference/androidx/work/Worker#setoutputdata)，该方法同样接收一个 [`Data`](https://developer.android.google.cn/reference/androidx/work/Data.html) 对象。您可以通过观测该任务的 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)<[`WorkStatus`](https://developer.android.google.cn/reference/androidx/work/WorkStatus.html)> 来获取输出的值。

例如，假设您有一个执行耗时计算的 [`Worker`](https://developer.android.google.cn/reference/androidx/work/Worker.html) 类，那么这个 [`Worker`](https://developer.android.google.cn/reference/androidx/work/Worker.html) 类的代码可能如下所示：

```java
// 定义 Worker 类：
public class MathWorker extends Worker {

    // 定义输入参数的键:
    public static final String KEY_X_ARG = "X";
    public static final String KEY_Y_ARG = "Y";
    public static final String KEY_Z_ARG = "Z";
    // ...和输出结果的 key:
    public static final String KEY_RESULT = "result";

    public MathWorker(
        @NonNull Context context,
        @NonNull WorkerParameters params) {
        super(context, params);
    }

    @Override
    public Worker.Result doWork() {
        // 获取输入参数（并指定缺省值）：
        int x = getInputData().getInt(KEY_X_ARG, 0);
        int y = getInputData().getInt(KEY_Y_ARG, 0);
        int z = getInputData().getInt(KEY_Z_ARG, 0);

        // ...执行数学计算...
        int result = myCrazyMathFunction(x, y, z);

        //...设置好输出结果，我们就大功告成了！
        Data output = new Data.Builder()
            .putInt(KEY_RESULT, result)
            .build();
        setOutputData(output);
        return Result.SUCCESS;
    }
}
```

欲创建任务并传入参数，您可以使用如下的代码：

```java
// 创建 Data 对象:
Data myData = new Data.Builder()
    // 我们需要传入三个整数：X、Y 和 Z
    .putInt(KEY_X_ARG, 42)
    .putInt(KEY_Y_ARG, 421)
    .putInt(KEY_Z_ARG, 8675309)
    // ... 并创建真正的 Data 对象:
    .build();

// ...然后使用这些参数来创建一个 OneTimeWorkRequest 并将其加入队列
OneTimeWorkRequest mathWork = new OneTimeWorkRequest.Builder(MathWorker.class)
        .setInputData(myData)
        .build();
WorkManager.getInstance().enqueue(mathWork);
```

返回的值可以在任务的 [`WorkStatus`](https://developer.android.google.cn/reference/androidx/work/WorkStatus.html) 中获取：

```java
WorkManager.getInstance().getStatusById(mathWork.getId())
    .observe(lifecycleOwner, status -> {
         if (status != null && status.getState().isFinished()) {
           int myResult = status.getOutputData().getInt(KEY_RESULT,
                  myDefaultValue));
           // ... 用输出结果做点什么 ...
         }
    });
```

如果您把任务串成链，那么上一个任务的输出就可以用来作为下一个任务的输入。
如果该任务链是两个 [`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.html) 的简单连接，那么第一个任务调用 [`setOutputData()`](https://developer.android.google.cn/reference/androidx/work/Worker.html#setOutputData(androidx.work.Data)) 来设置结果，而第二个任务则通过 [`getInputData()`](https://developer.android.google.cn/reference/androidx/work/Worker.html#getinputdata) 来获取该结果。
如果该任务链结构复杂，例如多个任务都将结果输出到同一个任务中，那么您可以在 [`OneTimeWorkRequest.Builder`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.Builder.html) 中定义一个 [`InputMerger`](https://developer.android.google.cn/reference/androidx/work/InputMerger.html) 用于指定不同任务返回有相同键的结果时应当采取什么操作。

## 其他资源

`WorkManager` 是 [Android Jetpack](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/A_Overview.md) 的一个架构组件。[Sunflower](https://github.com/googlesamples/android-sunflower) 范例应用展示了如何使用它们。


