# 使用能感知生命周期的组件来处理生命周期
> 原文链接：[Handling lifecycles with lifecycle-aware components  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/lifecycle)

能感知生命周期（lifecycle-aware）的组件会根据其他组件（例如 activity 和 fragment）的生命周期的状态改变而做出反应，从而帮助您编写更有条理、轻量且易于维护的代码。

一种常见的模式是在 acitivity 和 fragment 的生命周期方法中实现相关组件（dependent components）的行为，然而这种模式会导致混乱的代码和激增的错误。通过使用能感知生命周期的组件，您就能将相关组件的代码从生命周期方法中移出到这些组件自身中去。

[`android.arch.lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/package-summary.html) 包提供了让您构建能感知生命周期的组件的类和接口。

> **注意**：欲将 `android.arch.lifecycle` 导入到您的 Android 项目中，请参阅[向您的项目中添加 Android 架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_2_Adding_Components_to_your_Project.md)。

Android 框架中定义的大部分应用组件都附有相应的生命周期，而生命周期则是由操作系统、或是您的进程中运行的框架代码来管理。它们对于 Android 的正常工作来说是至关重要的，因此您的应用必须尊重其规律，否则就可能导致内存泄漏、甚至应用崩溃。

想象这样一个场景：我们有一个 acitivity 来在屏幕上显示设备的地理位置。一个常见的实现方式可能如下所示：

```java
class MyLocationListener {
    public MyLocationListener(Context context, Callback callback) {
        // ...
    }

    void start() {
        // 连接到系统的地理位置服务
    }

    void stop() {
        // 断开与系统的地理位置服务的连接
    }
}


class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    @Override
    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, (location) -> {
            // 更新 UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        myLocationListener.start();
        // 管理其他需要相应 activity 生命周期的组件
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
        // 管理其他需要相应 activity 生命周期的组件
    }
}
```

尽管这个例子看起来没什么问题，但在一个真实的应用中，您会发现最后有太多更新 UI 和其他组件相应当前生命周期状态的调用了。为了管理多个组件，您不得不向 [`onStart()`](https://developer.android.google.cn/reference/android/app/Activity.html#onStart()) 和 [`onStop()`](https://developer.android.google.cn/reference/android/app/Activity.html#onStop()) 等生命周期方法中添加大量的代码，这样就很难维护。

更糟的是，没人能保证这些组件会在 activity 或 fragment 停止前开始，尤其是在需要执行耗时较长的操作时，例如 [`onStart()`](https://developer.android.google.cn/reference/android/app/Activity.html#onStart()) 中的某些配置检查。这就会导致 [`onStop()`](https://developer.android.google.cn/reference/android/app/Activity.html#onStop()) 方法比 [`onStart()`](https://developer.android.google.cn/reference/android/app/Activity.html#onStart()) 更早结束的竞争危害（race condition），使这些组件存活得过久。

```java
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, location -> {
            // 更新 UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        Util.checkUserStatus(result -> {
            // 如果这个回调是在 activity 已经 结 jié 束 shù 后被调用呢？
            if (result) {
                myLocationListener.start();
            }
        });
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
    }
}
```

[`android.arch.lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/package-summary.html) 包提供了帮助您使用弹性而孤立的方法来应对这种问题的类和接口。

## 生命周期（Lifecycle）

[`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 是一个存储有一个组件（如 acitivity 或 fragment）生命周期状态的信息的类，并且它还允许其他对象观察这个状态。

[`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 使用两种主要的枚举类型来跟踪与其绑定的组件的生命周期状态：

### 事件（Event）

Android 框架和 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 类所分发的生命周期事件。这些事件一一对应到 acitivity 和 fragment 的回调事件。

### 状态（State）

[`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 对象所跟踪的组件的当前状态。

![Lifecycle-States](https://developer.android.google.cn/images/topic/libraries/architecture/lifecycle-states.png)

请将 State 想象成一个图（graph）的节点（node），而 Event 就是这些节点之间的边（edge）。

一个类可以通过给自己的方法添加注释来监测组件的生命周期状态。然后，通过调用 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 类的 [`addObserver()`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html#addObserver(android.arch.lifecycle.LifecycleObserver)) 方法、并传入观测者（observer）的实例，您就能添加这个观测者。如下例所示：

```java
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```

如上例所示，[`myLifecycleOwner`] 对象实现了 [`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html) 接口。我们下面就将解释。

## 生命周期所有者（LifecycleOwner）

[`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html) 是仅包含了一个方法的接口，用以表示某个类拥有 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html)。这唯一的 [`getLifecycle()`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html#getLifecycle()) 方法必须由那个类实现。另一方面，如果您试图管理的是整个应用进程的生命周期，请参阅 [`ProcessLifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/ProcessLifecycleOwner.html)。

该接口将 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 的所有权从各自的类（如 [`Fragment`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html) 和 [`AppCompatActivity`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html)）中抽象出来，并允许用户编写与其协作的组件。任何定制的应用类都可以实现该接口。


实现了 [`LifecycleObserver`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleObserver.html) 的接口的组件能够和实现了 [`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html) 的组件无缝衔接，因为后者可以提供一个生命周期，而前者可以注册一个观测。

继续用之前的地理位置跟踪应用作为例子，我们可以让 `MyLocationListener` 类实现 [`LifecycleObserver`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleObserver.html) 接口，并在 [`onCreate()`](https://developer.android.google.cn/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 中使用 activity 的 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 来初始化它。这样一来，`MyLocationListener` 就能自给自足，也就是说针对生命周期状态而做出反应的逻辑将在 `MyLocationListener` 而非 activity 中声明。让组件们各自保有其逻辑，有利于让 acitivity 和 fragment 的逻辑更易于管理。

```java
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, getLifecycle(), location -> {
            // 更新 UI
        });
        Util.checkUserStatus(result -> {
            if (result) {
                myLocationListener.enable();
            }
        });
  }
}
```

一个常见的用例是在 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 状态不对的时候避免执行特定的回调。譬如，如果回调在 activity 的状态已经保存后执行一个 FragmentTransaction 就会导致应用崩溃，所以我们必然不想执行这个回调。

为了照顾到这种用例，[`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 类允许其他对象查询当前的状态。

```java
class MyLocationListener implements LifecycleObserver {
    private boolean enabled = false;
    public MyLocationListener(Context context, Lifecycle lifecycle, Callback callback) {
       ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    void start() {
        if (enabled) {
           // 连接
        }
    }

    public void enable() {
        enabled = true;
        /* 注意⤵️ */
        if (lifecycle.getCurrentState().isAtLeast(STARTED)) {
            // 如果还没连接的话，就连接
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    void stop() {
        // 如果已经连接的话，就断开连接
    }
}
```

使用这样的实现方式，我们的 `LocationListener` 就是完全地能感知生命周期了。如果我们需要在别的 activity 或 fragment 里使用它，只需将其初始化就行了。所有的设置和拆解工作都会由这个类自己处理好。

如果一个库提供了需要和 Android 生命周期协调的类，我们推荐您使用能感知生命周期的组件。您的库的用户可以轻易地整合这些组件，而毋须手动地管理生命周期。

## 实现一个定制的 LifecycleOwner

26.1.0 及更新版本的支持库里的 Fragments 和 Activities 已经实现了 [`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html) 接口。

如果您想让某个定制的类变成 [`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html)，您可以使用 [`LifecycleRegistry`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleRegistry.html) 类，但是需要将事件转发到那个类。如下例所示：

```java
public class MyActivity extends Activity implements LifecycleOwner {
    private LifecycleRegistry mLifecycleRegistry;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mLifecycleRegistry = new LifecycleRegistry(this);
        mLifecycleRegistry.markState(Lifecycle.State.CREATED);
    }

    @Override
    public void onStart() {
        super.onStart();
        mLifecycleRegistry.markState(Lifecycle.State.STARTED);
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
}
```

## 能感知生命周期的组件的最佳实践

* 尽量让您的 UI 控制器（activity 和 fragment）别什么都自己扛着。它们不应试图获取自身的数据，而是应该通过 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html)，并观测一个 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象来在视图上针对变动做出反应。
* 试着写点数据驱动的 UI，并让您的 UI 控制器负责在数据变化的时候更新视图、或者把用户的行为通知给 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html)。
* 将您的数据逻辑放入 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 类。[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 应当充作您的 UI 控制器和应用其余部分的连接。请小心注意：获取数据（比如从网络下载）并不是 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 的责任，恰恰相反，[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 应当调用合适的组件去获取数据，再将结果提供给 UI 控制器。
* 使用[数据绑定](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_1_Overview.md)来在您的视图和 UI 控制器之间维持整洁的接口。这有利于让您的视图更具陈述性，而且让您在 UI 控制器中只需添加最少的更新逻辑。如果您更愿意使用 Java 语言来进行数据绑定，使用诸如 [Butter Knife](http://jakewharton.github.io/butterknife/) 的库可以避免八股代码而获得更好的抽象。
* 如果您的 UI 很复杂，请考虑创建一个 [presenter](http://www.gwtproject.org/articles/mvp-architecture.html#presenter) 类来处理 UI 修改。这可能会比较累人，但可以让您的 UI 组件更易于测试。
* 避免在您的 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 中直接引用 [`View`](https://developer.android.google.cn/reference/android/view/View.html) 或 [`Activity`](https://developer.android.google.cn/reference/android/app/Activity.html) 等 context。如果 `ViewModel` 比其 activity 还活得长久（比如配置变更），您的 activity 就内存泄漏了，很难被 GC 得当回收。

## 能感知生命周期的组件的用例

能感知生命周期的组件能让许多用例中的生命周期管理变得简单。以下是几个例子：

* 切换地理位置更新的精度。当您的应用可见时，启动高精度的地理位置更新；当您的应用进入后台时，切换成低精度的更新。[`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 就是一个能感知生命周期的组件，让您的应用在用户移动的时候自动更新 UI。
* 开始和停止视频缓冲。尽快开始视频缓冲，但把回放延迟到应用完全启动之后。您还可以使用这样的组件来在应用被销毁时停止视频缓冲。
* 开始和停止网络连接。当应用在前台时，开启使用网络连接的流媒体；当应用在后台时自动暂停。
* 暂停和恢复动画 drawable。当应用进入后台时，处理动画 drawable 的暂停；当应用回到前台时，恢复动画。

## 处理 onStop 事件

当 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 属于一个 [`AppCompatActivity`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html) 或 [`Fragment`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html)，[`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 的状态会变为 [`CREATED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#CREATED)，而 [`ON_STOP`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.Event.html#ON_STOP) 事件会在 [`AppCompatActivity`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html) 或 [`Fragment`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html) 的 [`onSaveInstanceState()`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onSaveInstanceState(android.os.Bundle)) 被调用时分发。

当一个 [`Fragment`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html) 或 [`AppCompatActivity`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html) 的状态在 [`onSaveInstanceState()`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onSaveInstanceState(android.os.Bundle)) 中保存时，直到 [`ON_START`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.Event.html#ON_START) 触发之前，其 UI 被认为是不可变动的。试图在状态保存后更改 UI 可能会导致您的应用产生不一致的导航状态，这就是为什么当应用在状态保存之后试图执行 [`FragmentTransaction`](https://developer.android.google.cn/reference/android/support/v4/app/FragmentTransaction.html) 时，[`FragmentManager`](https://developer.android.google.cn/reference/android/support/v4/app/FragmentManager.html) 会抛出异常。欲了解更多信息，请参阅 [`commit()`](https://developer.android.google.cn/reference/android/support/v4/app/FragmentTransaction.html#commit())。

[`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 提供了对这种边缘情况的开箱即用的支持：如果观测者所联系的 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 连 [`STARTED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#STARTED) 都不是的话，它就不会调用该观测者。其内部实现是先调用 [`isAtLeast()`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#isAtLeast(android.arch.lifecycle.Lifecycle.State)) 来决定是否要触发其观测者。

悲剧的是，[`AppCompatActivity`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html) 的 [`onStop()`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onStop()) 是在 [`onSaveInstanceState()`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onSaveInstanceState(android.os.Bundle)) 之后调用的，这导致了一段 UI 不允许改动、但 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 还没移入 [`CREATED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#CREATED) 的间隙。

为了防止这样的悲剧，`beta2` 及更低版本的 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 将状态标记为 [`CREATED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#CREATED) 但并不分发事件，这样一来任何检查当前状态的代码都会得到真正的值，尽管直到系统调用 [`onStop()`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onStop()) 之前事件都没被分发。

二次悲剧的是，这种解决方案有两个主要的问题：

* 在 23 及更低的 API 版本上，一个 activity 即使被另一个 activity 部分遮挡，Android 系统实际上也会保存其状态。换言之，Android 系统调用了 [`onSaveInstanceState()`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onSaveInstanceState(android.os.Bundle))，但并不一定调用 [`onStop()`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onStop())。这导致了一个可能很长的间隙，期间观测者仍然认为生命周期是活动的，但其实 UI 状态不能改动。
* 任何想要把类似行为暴露给 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 的类都需要实现 `beta2` 或更低版本的 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 所提供的变通方案。

> **注意**：为了让上述流程更简单、向后兼容性更好，从 **`1.0.0-rc1`** 版本开始，[**`Lifecycle`**](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 对象会被标记为 [**`CREATED`**](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#CREATED)，而 [**`ON_STOP`**](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.Event.html#ON_STOP) 会在 [**`onSaveInstanceState()`**](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onSaveInstanceState(android.os.Bundle)) 被调用时分发、毋须等待 [**`onStop`**](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity.html#onStop()) 方法被调用。这应该不太会影响您的代码，但您仍然需要了解，因为它和 26 及更高版本的 API 中 [**`Activity`**](https://developer.android.google.cn/reference/android/app/Activity.html) 方法的调用顺序并不一致。

## 其他资源

在[代码实验室](https://codelabs.developers.google.com/codelabs/android-lifecycles/#0)中试一试生命周期组件。

能感知生命周期的组件是 [`Android Jetpack`](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/A_Overview.md) 的一部分。[Sunflower](https://github.com/googlesamples/android-sunflower) 范例应用展示了如何使用它们。


