# 从 Firebase JobDispatcher 迁移
> 原文链接：[Migrating from Firebase JobDispatcher to WorkManager  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/workmanager/migrating-fb)

WorkManager 是一个用于调度和执行可延迟后台任务的 Android 库。它是 Firebase JobDispatcher 的推荐替代品。下文指南将引导您完成迁移 Firebase JobDispatcher 实现到 WorkManager 的过程。

## Gradle 配置

> **Note:** 迁移 Firebase JobDispatcher 的第一步是配置 WorkManager 的最新 gradle 依赖项。
>
> 要将 WorkManager 导入 Android 项目，请参阅 [WorkManager 发行说明](https://developer.android.google.cn/jetpack/androidx/releases/work#declaring_dependencies) 中的依赖项声明指示。

## 从 JobService 转变到 Workers

[`FirebaseJobDispatcher`](https://github.com/firebase/firebase-jobdispatcher-android/blob/e609dabf6cbd0fcc2451b8515f095cfbc3d9450a/jobdispatcher/src/main/java/com/firebase/jobdispatcher/FirebaseJobDispatcher.java) 引用
[`JobService`](https://github.com/firebase/firebase-jobdispatcher-android/blob/master/jobdispatcher/src/main/java/com/firebase/jobdispatcher/JobService.java) 的子类作为入口点，来定义需要完成的工作。您可以直接使用 `JobService` ，或者使用
[`SimpleJobService`](https://github.com/firebase/firebase-jobdispatcher-android/blob/master/jobdispatcher/src/main/java/com/firebase/jobdispatcher/SimpleJobService.java)。

`JobService` 实现一般是这样的：

```java
import com.firebase.jobdispatcher.JobParameters;
import com.firebase.jobdispatcher.JobService;

public class MyJobService extends JobService {
    @Override
    public boolean onStartJob(JobParameters job) {
        // 在这里做一些事情

        return false; // 问题来了：“到这之后任务还会继续运行吗？”
    }

    @Override
    public boolean onStopJob(JobParameters job) {
        return false; // 问题来了：“到这之后任务应该重试吗？”
    }
}
```

如果您使用的是 `SimpleJobService` ，就需要重写 `onRunJob()` ，其返回类型为 `@JobResult int` 。

关键的不同在于当您直接使用 `JobService` 时， `onStartJob()` 方法本身是在主线程调用的， 但您应当把其中的任务实现转移到后台线程运行。另一方面，如果您使用的是 `SimpleJobService` ，此 service 会直接在一个后台线程执行您的任务（无需自己实现线程）。

WorkManager 也有类似的概念。 WorkManager 的基本工作单元是
[`ListenableWorker`](https://developer.android.google.cn/reference/androidx/work/ListenableWorker)。 除此之外也有其它的 worker 子类，比如
[`Worker`](https://developer.android.google.cn/reference/androidx/work/Worker)，
[`RxWorker`](https://developer.android.google.cn/reference/androidx/work/RxWorker)，和 `CoroutineWorker`（当使用Kotlin协程时）。

### JobService 转换成 ListenableWorker

如果您原本直接用的是 `JobService`，那么就该相应地替换成 `ListenableWorker`。如果您用的是 `SimpleJobService`，就替换成 `Worker`。

我们用上面的例子 (`MyJobService`) 来看看怎么把它转换成一个 `ListenableWorker`。

```java
import android.content.Context;
import androidx.work.ListenableWorker;
import androidx.work.ListenableWorker.Result;
import androidx.work.WorkerParameters;
import com.google.common.util.concurrent.ListenableFuture;

class MyWorker extends ListenableWorker {

  public MyWorker(@NonNull Context appContext, @NonNull WorkerParameters params) {
    super(appContext, params);
  }

  @Override
  public ListenableFuture<ListenableWorker.Result> startWork() {
    // 做一些您自己的事情
    Data input = getInputData();

    // 返回一个 ListenableFuture<>
  }

  @Override
  public void onStopped() {
    // 在此做一些清除工作，因为任务已经处于终止状态
  }
}
```

WorkManager 中的基本工作单元是 `ListenableWorker`，会在主线程调用 `startWork()`，这和 `JobService.onStartJob()` 类似。 这里的 `MyWorker` 继承实现自 `ListenableWorker` 并返回一个 *异步* 发送任务完成信号的
[`ListenableFuture`](https://google.github.io/guava/releases/21.0-rc1/api/docs/com/google/common/util/concurrent/ListenableFuture.html) 实例。您应该在此选择自己的线程策略。

此处的 `ListenableFuture` 最终会返回一个 `ListenableWorker.Result` 类型，可能是 `Result.success()`， `Result.success(Data outputData)`，`Result.retry()`， `Result.failure()`，或 `Result.failure(Data outputData)`。欲知更多信息，请查看
[`ListenableWorker.Result`](https://developer.android.google.cn/reference/androidx/work/ListenableWorker.Result) 的参考文档。

`onStopped()` 被调用则表示 `ListenableWorker` 需要终止，要么因为限制条件不再满足（比如，网络变得不可用），要么因为 `WorkManager.cancel…()` 方法被调用。`onStopped()` 也可能因为某些原因被操作系统终止您的任务时而调用。

### SimpleJobService 转换成 Worker

当把 `SimpleJobService` 转换成上述 worker 时，就会像这样：

```java
import android.content.Context;
import androidx.work.Data;
import androidx.work.ListenableWorker.Result;
import androidx.work.Worker;
import androidx.work.WorkerParameters;

class MyWorker extends Worker {

  public MyWorker(@NonNull Context appContext, @NonNull WorkerParameters params) {
    super(appContext, params);
  }

  @Override
  public ListenableWorker.Result doWork() {
    // 做一些您自己的事情
    Data input = getInputData();

    // 返回一个 ListenableWorker.Result
    Data outputData = new Data.Builder()
        .putString(“Key”, “value”)
        .build();
    return Result.success(outputData);
  }

  @Override
  public void onStopped() {
    // 在此做一些清除工作，因为任务已经处于终止状态
  }
}
```

这里的 `doWork()` 返回一个 `ListenableWorker.Result` 实例来同步地通知任务已完成。这类似于在后台线程调度任务的 `SimpleJobService`。

## JobBuilder 转换成 WorkRequests

FirebaseJobBuilder 用 `Job.Builder` 来表达 `Job` 元数据。WorkManager 则用
[`WorkRequest`](https://developer.android.google.cn/reference/androidx/work/WorkRequest) 来满足相应的功能。

WorkManager 有两种类型的 `WorkRequest`：
[`OneTimeWorkRequest`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest) （一次性的）和
[`PeriodicWorkRequest`](https://developer.android.google.cn/reference/androidx/work/PeriodicWorkRequest)（周期性的）。

如果您目前用的是 `Job.Builder.setRecurring(true)`, 那么您应该转变成创建 `PeriodicWorkRequest`。否则（设为false的话），您就应当使用 `OneTimeWorkRequest`。

让我们看看用 `FirebaseJobDispatcher` 实现调度一个复杂的 `Job` 会是什么样子：

```java
Bundle input = new Bundle();
input.putString("some_key", "some_value");

Job myJob = dispatcher.newJobBuilder()
    //将要被调用的 JobService
    .setService(MyJobService.class)
    // 用来唯一标识 job
    .setTag("my-unique-tag")
    // 一次性 job
    .setRecurring(false)
    // 不要在设备重启后继续存在
    .setLifetime(Lifetime.UNTIL_NEXT_BOOT)
    // 从现在开始后的 0 到 60 秒内触发
    .setTrigger(Trigger.executionWindow(0, 60))
    // 不能重写同一标识的已存在的 job
    .setReplaceCurrent(false)
    // 用指数补偿的重试策略
    .setRetryStrategy(RetryStrategy.DEFAULT_EXPONENTIAL)
    // 执行 job 任务需要满足的限制条件
    .setConstraints(
        // 只在不按用量计费的网络下运行
        Constraint.ON_UNMETERED_NETWORK,
        // 只在设备充电时运行
        Constraint.DEVICE_CHARGING
    )
    .setExtras(input)
    .build();

dispatcher.mustSchedule(myJob);
```

而实现一个同样功能的 WorkManager，您需要：

- 构造能被 `Worker` 利用的输入数据。
- 用输入数据构造一个 `WorkRequest` 并设置和上述 `FirebaseJobDispatcher` 类似的限制条件。
- 将构造好的 `WorkRequest` 加入队列。

### 给 Worker 设置输入数据

`FirebaseJobDispatcher` 用 `Bundle` 来给 `JobService` 传递输入数据。而 WorkManager 则是用 [`Data`](https://developer.android.google.cn/reference/androidx/work/Data.Builder) ，因此实现就变成：

```java
import androidx.work.Data;
Data input = new Data.Builder()
    .putString("some_key", "some_value")
    .build();
```

### 给 Worker 设置约束条件

`FirebaseJobDispatcher` 用 [`Job.Builder.setConstaints(...)`](https://github.com/firebase/firebase-jobdispatcher-android/blob/master/jobdispatcher/src/main/java/com/firebase/jobdispatcher/Job.java#L287)
来给任务设置限制条件。而 WorkManager 则是用 
[`Constraints`](https://developer.android.google.cn/reference/androidx/work/Constraints)。

```java
import androidx.work.Constraints;
import androidx.work.Constraints.Builder;
import androidx.work.NetworkType;

Constraints constraints = new Constraints.Builder()
    // 设置 Worker 需要网络连接
    .setRequiredNetworkType(NetworkType.CONNECTED)
    // 设置为需要设备处于充电状态
    .setRequiresCharging(true)
    .build();
```

### 创建 WorkRequest （一次性（OneTime）或周期性（Periodic）类型）

您应该用 [`OneTimeWorkRequest.Builder`](https://developer.android.google.cn/reference/androidx/work/OneTimeWorkRequest.Builder) 和 [`PeriodicWorkRequest.Builder`](https://developer.android.google.cn/reference/androidx/work/PeriodicWorkRequest.Builder) 来创建 `OneTimeWorkRequest` 和 `PeriodicWorkRequest`。

创建一个类似于上述 `Job` 的 `OneTimeWorkRequest` 您应该像如下这样做：

```java
import androidx.work.BackoffCriteria;
import androidx.work.Constraints;
import androidx.work.Constraints.Builder;
import androidx.work.NetworkType;
import androidx.work.OneTimeWorkRequest;
import androidx.work.OneTimeWorkRequest.Builder;
import androidx.work.Data;

// 定义限制条件 (如上所述)
Constraints constraints = ...
OneTimeWorkRequest request =
    // 声明要执行的任务
    new OneTimeWorkRequest.Builder(MyWorker.class)
        // 给 ListenableWorker 设置输入数据
        .setInputData(inputData)
        // 设置任务开始时间延后 60 秒
        .setInitialDelay(60, TimeUnit.SECONDS)
        // 设置重试时的补偿策略
        .setBackoffCriteria(BackoffCriteria.EXPONENTIAL, 30000, TimeUnit.MILLISECONDS)
        // 设置额外的限制条件
        .setConstraints(constraints)
        .build();
```

此处关键的不同点在于 WorkManager 的任务总会自动地维持到设备重启以后。

如果您想创建 `PeriodicWorkRequest` ，您可以这样做：

```java
import androidx.work.BackoffCriteria;
import androidx.work.Constraints;
import androidx.work.Constraints.Builder;
import androidx.work.NetworkType;
import androidx.work.PeriodicWorkRequest;
import androidx.work.PeriodicWorkRequest.Builder;
import androidx.work.Data;

// 定义限制条件 (如上所述)
Constraints constraints = ...

PeriodicWorkRequest request =
    // 每 15 分钟执行一次 MyWorker
    new PeriodicWorkRequest.Builder(MyWorker.class, 15, TimeUnit.MINUTES)
        // 给 ListenableWorker 设置输入数据
        .setInputData(input)
        . // 一些其余的 setter
        .build();
```

## 调度任务

现在您已经定义好了 `Worker` 和 `WorkRequest`，准备来调度任务吧。

每个由 `FirebaseJobDispatcher` 定义的 `Job` 都有一个 `tag` 用来进行 *唯一标识* 。您也可以调用 `setReplaceCurrent` 方法来告知调度器（scheduler），把已有的 `Job` 实例替换进来。

```java
Job myJob = dispatcher.newJobBuilder()
    // 将会被调用的 JobService
    .setService(MyJobService.class)
    // 唯一标识 job
    .setTag("my-unique-tag")
    // 设置不要被同一 tag 的 job 替换覆盖
    .setReplaceCurrent(false)
    // 一些其余的 setter
    // ...

dispatcher.mustSchedule(myJob);
```

当使用 WorkManager 时，你可以通过 `enqueueUniqueWork()` 和 `enqueueUniquePeriodicWork()` API（对应于您创建的 `OneTimeWorkRequest` 和 `PeriodicWorkRequest`）实现相同的结果。欲知更多信息，请查看
[`WorkManager.enqueueUniqueWork()`](https://developer.android.google.cn/reference/androidx/work/WorkManager#enqueueUniqueWork(java.lang.String,%20androidx.work.ExistingWorkPolicy,%20androidx.work.OneTimeWorkRequest)) 和
[`WorkManager.enqueueUniquePeriodicWork()`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html#enqueueUniquePeriodicWork(java.lang.String,%20androidx.work.ExistingPeriodicWorkPolicy,%20androidx.work.PeriodicWorkRequest)) 的参考文档。

代码实现如下：

```java
import androidx.work.ExistingWorkPolicy;
import androidx.work.OneTimeWorkRequest;
import androidx.work.WorkManager;

OneTimeWorkRequest workRequest = // 自行构造好 WorkRequest
WorkManager.getInstance()
    // 用 ExistingWorkPolicy.REPLACE 策略来取消和删除相同标识的
    // 等待中（未完成）的任务。然后再插入新的特定任务。
    .enqueueUniqueWork("my-unique-name", ExistingWorkPolicy.KEEP, workRequest);
```

> **注意：** `Job` 在 FirebaseJobDispatcher 中的 tag 对应到 `WorkRequest` 中就是 `name`。

## 取消任务

在 `FirebaseJobDispatcher` 中你可以这样取消任务：

```java
dispatcher.cancel("my-unique-tag");
```

而在 WorkManager 中你应该这样：

```java
import androidx.work.WorkManager;
WorkManager.getInstance().cancelUniqueWork("my-unique-name");
```

## 初始化 WorkManager

WorkManager 需要各个应用程序自行初始化，通常是用 `ContentProvider` 或者在 `Application.onCreate()` 中实现。

一般来说是用 `ContentProvider` 来初始化 WorkManager。然而，在关于线程池大小的默认情况和给定时间内可被调度的 worker 数量之间存在一些微妙的差异。因此你可能需要自定义 WorkManager。

通常这种自定义以
[`WorkManager.initialize()`](https://developer.android.google.cn/reference/androidx/work/WorkManager.html#initialize(android.content.Context,%20androidx.work.Configuration)) 方法结束。你可以自定义后台 `Executor` 来执行 `Worker`，并且
[`WorkerFactory`](https://developer.android.google.cn/reference/androidx/work/WorkerFactory) 可用来构造 `Workers`。（`WorkerFactory` 在依赖注入的 context 中非常有用）。请阅读关于这个方法的文档以确保你能终止 WorkManager 的自动初始化。

欲知更多信息，请参阅关于 `initialize()` 和
[`Configuration.Builder`](https://developer.android.google.cn/reference/androidx/work/Configuration.Builder) 的文档。
