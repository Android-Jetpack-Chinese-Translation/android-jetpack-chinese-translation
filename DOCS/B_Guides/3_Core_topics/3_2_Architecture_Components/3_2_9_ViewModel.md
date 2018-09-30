# ViewModel 概览
> 原文链接：[ViewModel Overview  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/viewmodel)

[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 是一种被设计为通过能感知生命周期的方式来存储和管理 UI 相关的数据的类。它让数据能存活过配置变更（如屏幕旋转）而不被杀死。

> **注意**：欲将 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 导入您的 Android 项目，请参阅[向您的项目中添加 Android 架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_2_Adding_Components_to_your_Project.md)。

Android 框架管理着 activity 和 fragment 等 UI 控制器的生命周期，因此可能决定销毁或重建某个 UI 控制器来相应特定的用户操作或设备事件，而这些操作和事件是您完全无法掌控的。

如果系统销毁或重建了某个 UI 控制器，您在其中存储的任何与 UI 相关的临时性数据都将丢失。例如，您的应用可能在某个 activity 中包含了一个用户列表。当这个 activity 因配置变更而被重建时，新的 activity 需要重新获取这个用户列表。对于简单的数据，activity 可以利用 [`onSaveInstanceState()`](https://developer.android.google.cn/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)) 方法，并在 [`onCreate()`](https://developer.android.google.cn/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 时从 bundle 中重建这些数据。然而，这种方法显然只适合少量可以序列化/反序列化的数据，而不适用于可能数量巨大的数据，如一个列表的用户或位图。

另一个问题是，UI 控制器常常调用耗时的异步方法。UI 控制器需要管理这些调用，并确保系统在销毁自己时将这些数据清理掉，以免造成内存泄漏。这种管理需要大量精力维护，而且对象在因配置变更而重建时，往往浪费资源于重新执行它刚刚已经调用完毕的方法。

Activity 和 fragment 等 UI 控制器应当主要被用于展示 UI 数据、对用户操作进行反应、或者处理与操作系统的通讯（如权限请求），要求 UI 控制器同时肩负从数据库或网络加载数据等责任会使其太臃肿。把过多责任分配给 UI 控制器可能导致单个类试图自己处理整个应用的全部工作、而不是把工作委托给其他类完成，还会让测试变得极其困难。

为了让人生轻松高效一点，不妨从 UI 控制器的业务中分离出 view 的数据所有权。

## 实现一个 ViewModel

架构组件为负责准备 UI 数据的 UI 控制器提供了 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 这个助手类，它能在配置变更时自动保全自身，因而能立即将其数据提供给下一个 activity 或 fragment 实例。例如，如果您需要在应用中展示一个用户的列表，请务必把获取和存储数据的责任交给一个 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 而非 activity 或 fragment，如下例所示：

```java
public class MyViewModel extends ViewModel {
    private MutableLiveData<List<User>> users;
    public LiveData<List<User>> getUsers() {
        if (users == null) {
            users = new MutableLiveData<List<User>>();
            loadUsers();
        }
        return users;
    }

    private void loadUsers() {
        // 执行一个获取用户的异步操作
    }
}
```

之后，您就能如下所示地从 activity 中获取这个列表：

```java
public class MyActivity extends AppCompatActivity {
    public void onCreate(Bundle savedInstanceState) {
        // 当系统第一次调用 activity 的 onCreate() 时，创建一个 ViewModel
        // 重建的 activity 都会获得相同的那个由第一个 activity 所创建的 MyViewModel 实例

        MyViewModel model = ViewModelProviders.of(this).get(MyViewModel.class);
        model.getUsers().observe(this, users -> {
            // 更新 UI
        });
    }
}
```

如果 activity 被重建，它将获得相同的那个由第一个 activity 所创建的 MyViewModel 实例。当一个 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 对象的 activity 所有者结束后，Android 框架会调用其 [`onCleared()`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html#onCleared()) 方法，以便让它自行清理资源。

> **注意**：一个 [**`ViewModel`**](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 决不能引用视图（view）、[**`Lifecycle`**](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html)、或任何可能持有 activity context 引用的类。

[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 对象被设计为比特定视图或 [`LifecycleOwners`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html) 的实例有更长的生命周期。这种设计也意味着您可以更轻松地编写覆盖一个 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 的测试，因为它不关心视图和 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 对象。[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 对象可以含有 [`LifecycleObservers`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleObserver.html)（如 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)），然而决不能观测其变化。如果 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 需要 [`Application`](https://developer.android.google.cn/reference/android/app/Application.html) context，例如在寻找某个系统服务时，它可以继承 [`AndroidViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/AndroidViewModel.html) 类，并提供一个接受 [`Application`](https://developer.android.google.cn/reference/android/app/Application.html) 参数的构造器，因为 [`Application`](https://developer.android.google.cn/reference/android/app/Application.html) 也继承了 [`Context`](https://developer.android.google.cn/reference/android/content/Context.html)。

## ViewModel 的生命周期

[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 对象被限定为在获取自己时传入 [`ViewModelProvider`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModelProvider.html) 的 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html)。在这个 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 永久死去之前，[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 都将留存在内存中：比如 activity 的 finish，或者 fragment 的 detach。

图一展示了一个 activity 在旋转和结束的过程中所经历的各种生命周期状态，以及与其关联的 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 的生命周期。本图展示了 activity 的情况，而同样的基本状态也适用于 fragment 的生命周期。

![图一](https://developer.android.google.cn/images/topic/libraries/architecture/viewmodel-lifecycle.png)

一般而言，您可以在系统第一次调用某个 activity 对象的 [`onCreate()`](https://developer.android.google.cn/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 方法时请求特定的 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html)。系统可能会在整个 activity 的生命中多次调用 [`onCreate()`](https://developer.android.google.cn/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 方法，例如当设备的屏幕旋转时。[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 从您第一次请求它开始存在，直到 activity 被结束并销毁。

## 在 fragment 之间分享数据

一个很常见的场景是，在同一个 activity 里的两个或更多 fragment 需要在彼此之间通信。想象这样一个大纲-细节模式（master-detail）的常见用例：您使用一个 fragment 来提供一个列表供用户选择其中的某个项目，同时用另一个 fragment 展示被选中的项目的详细内容。这个例子肯定不简单，因为两个 fragment 需要定义一些接口，而且它们的宿主 activity 必须将两者联系在一起。此外，两个 fragment 还都得处理好另一个 fragment 还没就绪的情况。

这个常见的痛点可以通过使用 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 来解决。上述的两个 fragment 可以分享同一个 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html)，让它使用宿主 activity 的作用域来处理通信。如下例所示：

```java
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

    public void select(Item item) {
        selected.setValue(item);
    }

    public LiveData<Item> getSelected() {
        return selected;
    }
}


public class MasterFragment extends Fragment {
    private SharedViewModel model;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        itemSelector.setOnClickListener(item -> {
            model.select(item);
        });
    }
}

public class DetailFragment extends Fragment {
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SharedViewModel model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        model.getSelected().observe(this, item -> {
           // 更新 UI.
        });
    }
}
```

注意：两个 fragment 使用 [`getActivity()`](https://developer.android.google.cn/reference/android/app/Fragment.html#getActivity()) 来获取 [`ViewModelProvider`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModelProvider.html)，结果使得它们接收到相同的 `SharedViewModel` 实例，而后者有着和宿主 activity 相同的作用域。

这种方案有着下列好处：

* 宿主 Activity 什么都不用做，也不需要关心这种通信。
* 除了 `SharedViewModel` 的协议，Fragment 无须知道彼此。即使其中一个消失了，另一个也能照常工作。
* 每个 fragment 有其自己的生命周期，且不受另一个 fragment 的生命周期的影响。即使一个 fragment 取代了另一个，UI 仍能继续照常工作。

## 用 ViewModel 来替代[加载器（loader）](https://developer.android.google.cn/guide/components/loaders?hl=zh-cn)

为了让应用 UI 的数据和数据库保持同步，我们往往需要使用 [`CursorLoader`](https://developer.android.google.cn/reference/android/content/CursorLoader.html) 等加载器。您可以使用 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 和其他一些类来替代加载器。使用 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 可以把您的 UI 控制器从数据加载的操作中分离出来，也就意味着您的类之间有着更少的强引用（strong reference）。

在一个常见的使用加载器的方案中，某个应用可能使用 [`CursorLoader`](https://developer.android.google.cn/reference/android/content/CursorLoader.html) 来观测数据库的内容。当数据库内的一个值变化时，加载器自动触发数据的重载并更新 UI：

![**图二**：使用加载器来加载数据](https://developer.android.google.cn/images/topic/libraries/architecture/viewmodel-loader.png)

[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 能和 [`Room`](https://developer.android.google.cn/topic/libraries/architecture/room.html) 以及 [`LiveData`](https://developer.android.google.cn/topic/libraries/architecture/livedata.html) 协作，用于替换掉加载器。[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 保证了数据能活过配置变更，[`Room`](https://developer.android.google.cn/topic/libraries/architecture/room.html) 在数据库发生变化时通知您的 [`LiveData`](https://developer.android.google.cn/topic/libraries/architecture/livedata.html)，而 [`LiveData`](https://developer.android.google.cn/topic/libraries/architecture/livedata.html) 则使用修正后的新数据来更新 UI。

![**图三**：使用 ViewModel 来加载数据](https://developer.android.google.cn/images/topic/libraries/architecture/viewmodel-replace-loader.png)

## 其他资源

[这个博客](https://codelabs.developers.google.com/codelabs/android-persistence/#0) 描述了如何使用 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 和 [`LiveData`](https://developer.android.google.cn/topic/libraries/architecture/livedata.html) 来替换掉 [`AsyncTaskLoader`](https://developer.android.google.cn/reference/android/content/AsyncTaskLoader.html)。

随着您的数据变得越来越复杂，您可能选择用一个单独的类来只负责数据加载。[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 的目的是把 UI 控制器的数据封装起来，从而活过配置变更。欲了解更多有关在配置变更前后、对数据进行加载、持久化和管理的内容，请参阅[保存 UI 状态](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_11_Saving_States.md)。

[应用架构指南](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/B_Get_started/2_Guide_to_app_architecture.md) 提供了如何构建一个 repository 类来处理这些功能的建议。

[`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 是一个 [Android Jetpack](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/A_Overview.md) 的架构组件。[Sunflower](https://github.com/googlesamples/android-sunflower) 范例应用展示了如何使用它们。

