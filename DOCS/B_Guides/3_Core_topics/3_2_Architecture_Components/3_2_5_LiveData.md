# LiveData 概览
> 原文链接：[LiveData overview  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/lifecycle)

[`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 是一个可观测数据的容器类。与普通的可观测类不同，LiveData 能感知生命周期，也就意味着它合乎 activity、fragment 或 service 等其他应用组件的生命周期。因此，LiveData 确保只在这些可观测的应用组件处于活动状态的时候才会更新它们。

> **注意**：欲将 LiveData 组件导入您的 Android 项目，请参阅[向您的项目中添加 Android 架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_2_Adding_Components_to_your_Project.md)。

LiveData 认为，用 [`Observer`](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html) 表示的观测者处于 [`STARTED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#STARTED) 或 [`RESUMED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#RESUMED) 的时候是活动的。LiveData 把数据的更新只通知活动的观测者，而处于非活动状态的那些注册监视 LiveData 对象的观测者则不会被通知。

您可以把一个观测者与实现了 [`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html) 接口的对象配对注册。这种关系允许观测者在相关的 [`LifeCycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 对象变成 [`DESTROYED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#DESTROYED) 的时候被移除。这对于 activity 和 fragment 而言特别有用，因为它们可以安全地观测 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象，而无须担心内存泄漏——activity 和 fragment 在生命周期结束的时候会被立即取消订阅。

欲了解更多有关使用 LiveData 的内容，请参阅下文的“使用 LiveData 对象”。

### 使用 LiveData 的好处

使用 LiveData 有下列好处：

#### 确保您的 UI 和数据的状态保持一致

LiveData 符合观测者的样式。当生命周期的状态产生变化时，它就会通知 [`Observer`](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html) 对象。您可以将更新 UI 的代码放到这些 `Observer` 的观测者对象中，从而让代码更稳健。您的观测者会在每一次应用数据产生变化时更新 UI，这样您就无须在数据变化的每个位置都添加更新 UI 的代码。

#### 没有内存泄漏

观测者受 [`LifeCycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 的作用域限制，而且还会在与其关联的生命周期被销毁后自动清理自己。

#### 不会因为 activity 停止而导致应用崩溃
	
如果观测者的生命周期处于非活动状态，例如在回退栈（back stack）里的 activity，那么它就不会收到任何 LiveData 的事件。

#### 无须手动管理生命周期

UI 组件只观测相关的数据，无须手动开始或停止。LiveData 会自动处理好这一切，因为它能在观测的同时，感知到相关的生命周期状态变动。

#### 总是最新的数据

即使一个生命周期进入非活动状态，在它恢复活动状态的时候仍能接收最新的数据。例如，一个后台的 activity 在回到前台的时刻就会立即接收到最新的数据。

#### 安稳度过配置变更（configuration changes）

如果一个 activity 或 fragment 因屏幕旋转等配置变更而被重建，它仍能立即接收到最新的可用的数据。

#### 分享资源

您可以使用单例模式来继承一个 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象来把系统服务包装起来，因而使其能够分享给整个应用。`LiveData` 对象只需连接到系统服务一次，之后任何需要这些资源的观测者都可以直接观测 `LiveData` 对象。欲了解更多信息，请参阅下文中的“继承 LiveData”。

## 使用 LiveData 对象
	
欲使用 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象，请按照下述步骤操作：

1. 创建一个 `LiveData` 的实例来存储某个特定类型的数据。一般而言，您需要在 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 中这么做。
2. 创建一个 [`Observer`](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html) 对象，并实现其 [`onChanged()`](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html#onChanged(T)) 方法，用于控制在 `LiveData` 的数据更新时应执行的操作。一般而言，您需要在 activity 或 fragment 等 UI 控制器中创建一个 `Observer` 对象。
3. 使用 [`observer()`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#observe(android.arch.lifecycle.LifecycleOwner,%0Aandroid.arch.lifecycle.Observer%3CT%3E)) 方法将 `Observer` 对象关联到 `LiveData` 对象上。该方法需要传入一个 [`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html) 对象。这样一来，`Observer` 对象就订阅了 `LiveData` 对象，因而能在后者产生变动的时候收到通知。一般而言，您需要在 activity 或 fragment 等 UI 控制器中关联 `Observer` 对象。

> **注意**：您也可以调用 [`observeForever(Observer)`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#observeForever(android.arch.lifecycle.Observer%3CT%3E)) 来注册一个观测者而无须提供相关的 [`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html)。在这种情况下，观测者被认为永远处于活动状态，因此永远会在数据变化时接收到通知。要移除这些观测者，您可以调用 [`removeObserver(Observer)`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#removeObserver(android.arch.lifecycle.Observer%3CT%3E)) 方法。

当您更新 `LiveData` 所存储的数据的值时，只要其关联的 `LifecycleOwner` 处于活动状态，它就将触发所有的观测者。

综上所述，LiveData 允许 UI 控制器中的观测者注册订阅，当其存储的数据产生变化时，UI 会自动执行相应的更新。

### 创建 LiveData 对象

LiveData 可以包装任何类型的数据，甚至包括 [`List`](https://developer.android.google.cn/reference/java/util/List.html) 等实现了 [`Collections`](https://developer.android.google.cn/reference/java/util/Collections.html) 接口的类。一般而言，[`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象是存储在一个 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 中的，并可以通过一个 getter 方法来获取。如下例所示：

```java
public class NameViewModel extends ViewModel {

    // 创建一个存储 String 的 LiveData 对象
    private MutableLiveData<String> mCurrentName;

    public MutableLiveData<String> getCurrentName() {
        if (mCurrentName == null) {
            mCurrentName = new MutableLiveData<String>();
        }
        return mCurrentName;
    }

// ViewModel 的其余部分……
}
```

在初始状态，`LiveData` 对象中的数据是没有被赋值的。

> **注意**：请务必将更新 UI 的 **LiveData** 对象存储在 **ViewModel** 对象中，而不是 activity 或 fragment。原因如下：
> * 避免过于臃肿的 activity 和 fragment。现在这些 UI 控制器负责展示数据，而非存储数据的状态。
> * 将 **LiveData** 实例从特定的 activity 或 fragment 中解耦出来，并允许 **LiveData** 存活过配置变更。

欲了解更多有关 `ViewModel` 的好处和使用方法，请参阅 [ViewModel](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_9_ViewModel.md)。

### 观测 LiveData 对象

在大多数情况下，您应当在某个应用组件的 [`onCreate()`](https://developer.android.google.cn/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 方法中开始对 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象的观测，原因如下：

* 确保系统不会因为执行 activity 或 fragment 的 [`onResume()`](https://developer.android.google.cn/reference/android/app/Activity.html#onResume()) 方法而导致重复的调用。
* 确保 activity 或 fragment 能在进入活动状态时尽早获得用于展示 UI 的数据。当某个应用组件进入 [`STARTED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#STARTED) 状态的那一刻，它就能接收到所观测的 `LiveData` 对象的最新数据。当然，这只有在 `LiveData` 已经被设置好的情况下才能观测。

一般而言，LiveData 只在数据变化的情况下才会将更新通知给活动状态的观测者。然而这有一个例外：观测者在从非活动状态进入活动状态的时候，也能收到更新。更进一步讲，如果观测者又一次从非活动状态进入活动状态，那么只有当数据在这期间又发生了变化，观测者才会收到更新。

如下的示例代码展示了如何开始观测一个 `LiveData` 对象：

```java
public class NameActivity extends AppCompatActivity {

    private NameViewModel mModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // 其他设置 activity 的代码……

        // 获取 ViewModel
        mModel = ViewModelProviders.of(this).get(NameViewModel.class);


        // 创建用于更新 UI 的观测者
        final Observer<String> nameObserver = new Observer<String>() {
            @Override
            public void onChanged(@Nullable final String newName) {
                // Update the UI, in this case, a TextView.
                mNameTextView.setText(newName);
            }
        };

        // 将这个 activity 作为 LifecycleOwner 参数，和观测者一并传入，开始观测该 LiveData
        mModel.getCurrentName().observe(this, nameObserver);
    }
}
```

在 [`observe()`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#observe(android.arch.lifecycle.LifecycleOwner,%0Aandroid.arch.lifecycle.Observer%3CT%3E)) 被传入 `nameObserver` 参数并调用后，[`onChanged()`](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html#onChanged(T)) 立即被触发，用于提供 `mCurrentName` 所存储的数据的最新的值。如果 `LiveData` 对象还没有为 `mCurrentName` 设置一个值，那么 `onChanged()` 就不会被调用。

### 更新 LiveData 对象

LiveData 没有提供公开可用的更新其数据的方法；[`MutableLiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html) 类则把 [`setValue(T)`](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html#setValue(T)) 和 [`postValue(T)`](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html#postValue(T)) 方法公开暴露出来，因此您若需要编辑 LiveData 中的数据，就必须使用这两个方法。一般而言，`MutableLiveData` 是存储在 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 中的，而 `ViewModel` 只把可变的 `LiveData` 对象暴露给观测者。

当您已经设置好观测者的关系后，就可以更新 `LiveData` 对象中的数据了。如下面的例所示，用户点击按钮时所有观测者都被通知：

```java
mButton.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        String anotherName = "Naco Siren";
        mModel.getCurrentName().setValue(anotherName);
    }
});
```

在该例中调用 `setValue(T)` 会导致观测者使用新的值 `Naco Siren` 去调用其 [`onChanged`](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html#onChanged(T)) 方法。该例展示的是用户点击按钮的情况，但还有更多使用 `setValue()` 或 `postValue()` 的情形，比如对网络请求或者数据库完成加载做出反应：无论是哪种情况，`setValue()` 或 `postValue()` 的调用都会通知观测者去更新 UI。

> **注意**：在主线程中，您必须调用 [`setValue(T)`](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html#setValue(T)) 来更新 **LiveData** 对象。如果代码是在工作线程中执行的，那么您可以使用 [`postValue(T)`](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html#postValue(T)) 来更新它。

### 和 Room 一起使用 LiveData

[Room](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md) 数据持久化库支持可观测的查询（observable queries），并返回 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData) 对象。可观测的查询是数据库访问对象（Database Access Object，DAO）的一部分。

当数据库更新时，Room 会自动产生所有必要的代码来更新 `LiveData` 对象。在必要的时候，这些产生的代码会异步地运行在一个后台线程上。这种模式有利于保持 UI 展示的数据与数据库保持一致。欲了解更多有关 Room 和 DAO 的内容，请参阅 [Room 持久化库](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md)。

## 继承 LiveData

如果观测者的生命周期不处于 [`STARTED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#STARTED) 或 [`RESUMED`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#RESUMED)，那么 LiveData 就认为它处于非活动的状态。如下的示例代码展示了如何继承 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData) 类：

```java
public class StockLiveData extends LiveData<BigDecimal> {
    private StockManager mStockManager;

    private SimplePriceListener mListener = new SimplePriceListener() {
        @Override
        public void onPriceChanged(BigDecimal price) {
            setValue(price);
        }
    };

    public StockLiveData(String symbol) {
        mStockManager = new StockManager(symbol);
    }

    @Override
    protected void onActive() {
        mStockManager.requestPriceUpdates(mListener);
    }

    @Override
    protected void onInactive() {
        mStockManager.removeUpdates(mListener);
    }
}
```

该例所展示的价格监听器的实现包含了下列方法：

* 当 `LiveData` 有了活动状态的观测者时，其 [`onActive`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#onActive()) 方法被调用。这代表着您需要在这个方法中开始对股价的监听。
* 当 `LiveData` 没有任何处于活动状态的观测者时，其 [`onInactive`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#onInactive()) 方法被调用。由于没有观测者在监听了，您就没必要让 `StockManager` 继续保持连接状态。
* [`setValue(T)`](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html#setValue(T)) 方法更新 `LiveData` 示例的值，并把更新通知给每个处于活动状态的观测者。

您可以这样使用 `StockLiveData`：

```java
public class MyFragment extends Fragment {
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        LiveData<BigDecimal> myPriceListener = ...;
        myPriceListener.observe(this, price -> {
            // 更新 UI
        });
    }
}
```

[`observe()`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#observe(android.arch.lifecycle.LifecycleOwner,%0Aandroid.arch.lifecycle.Observer%3CT%3E)) 方法将 fragment 作为 [`LifecycleOwner`](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html) 作为第一个参数。这样做是为了表明该观测者受限于该 `LifecycleOwner` 的生命周期 [`Lifecycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html)。因此：

* 如果 `Lifecycle` 对象并不处于活动状态，那么即使数据发生了变化，观测者也不会被通知。
* 当 `Lifecycle` 对象被销毁后，观测者也会被自动移除。

`LiveData` 对象能感知生命周期这一点也意味着，您可以将其分享给多个 activity、fragment 和 service。为了让示例代码简单一些，您可以用单例模式这样实现 `LiveData`：

```java
public class StockLiveData extends LiveData<BigDecimal> {
    private static StockLiveData sInstance;
    private StockManager mStockManager;

    private SimplePriceListener mListener = new SimplePriceListener() {
        @Override
        public void onPriceChanged(BigDecimal price) {
            setValue(price);
        }
    };

    @MainThread
    public static StockLiveData get(String symbol) {
        if (sInstance == null) {
            sInstance = new StockLiveData(symbol);
        }
        return sInstance;
    }

    private StockLiveData(String symbol) {
        mStockManager = new StockManager(symbol);
    }

    @Override
    protected void onActive() {
        mStockManager.requestPriceUpdates(mListener);
    }

    @Override
    protected void onInactive() {
        mStockManager.removeUpdates(mListener);
    }
}
```

……并在 fragment 中这样使用它：

```java
public class MyFragment extends Fragment {
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        StockLiveData.get(getActivity()).observe(this, price -> {
            // 更新 UI 
        });
    }
}
```

`MyPriceListener` 示例可供多个 fragment 和 activity 观测，而 LiveData 只会在其中至少一个或多个处于可见的活动状态时，才会连接到系统服务。

## 变换 LiveData

有时您可能会想要在 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象把数据更新分发给观测者之前对数据进行更改，或者想要根据另一个 `LiveData` 的值来返回一个不同的 `LiveData` 实例。[`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/package-summary.html) 包中的 [`Transformations`](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html) 类提供了若干辅助函数来帮助您处理这些情况。

* [`Transformations.map()`](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html#map(android.arch.lifecycle.LiveData%3CX%3E,%20android.arch.core.util.Function%3CX,%20Y%3E))
	
	把一个函数变换应用到 `LiveData` 对象所存储的值，并将结果向下传递。
	
	```java
	LiveData<User> userLiveData = ...;
	LiveData<String> userName = Transformations.map(userLiveData, 	user -> {
  	    user.name + " " + user.lastName
	});
	```
	
* [`Transformations.switchMap()`](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html#switchMap(android.arch.lifecycle.LiveData%3CX%3E,%20android.arch.core.util.Function%3CX,%20android.arch.lifecycle.LiveData%3CY%3E%3E))
	
	与 `map()` 相似地，把一个函数变换应用到 `LiveData` 对象所存储的值，并将结果拆包后向下传递。该函数变换必须返回一个 `LiveData` 对象，如下例所示：
	
	```java
	private LiveData<User> getUser(String id) {
  		...;
	}

	LiveData<String> userId = ...;
	LiveData<User> user = Transformations.switchMap(userId, id -> 	getUser(id) );
	```

您可以使用上述变换函数来在观测者的整个生命周期中传递信息。如果观测者没有监视返回的 `LiveData` 对象，那么变换函数的计算就不会被执行。由于变换是惰性求值的，生命周期相关的行为也会被隐式地传递，无须任何显式调用或依赖。

如果您在 [`ViewModel`](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html) 对象中需要一个 `LifeCycle` 对象，那么变换可能是个更好的解决方案。譬如，假设您有一个 UI 组件来接收地址并返回其邮政编码，那么您也许会这样为该组件实现一个拿义务的 `ViewModel`：

```java
class MyViewModel extends ViewModel {
    private final PostalCodeRepository repository;
    public MyViewModel(PostalCodeRepository repository) {
       this.repository = repository;
    }

    private LiveData<String> getPostalCode(String address) {
       // 别这么干⬇️
       return repository.getPostCode(address);
    }
}
```

如果这么做的话，UI 组件每次调用 `getPostalCode()` 时都需要从之前的 `LiveData` 取消订阅，然后再注册新实例的订阅。更糟的是，如果 UI 组件被重建的话，它将触发一次新的对 `repository.getPostCode()` 的调用，而不是利用上一次调用的结果。

与之相反，您应当把邮政编码的查询实现为一个从地址输入到 LiveData 的变换，如下例所示：

```java
class MyViewModel extends ViewModel {
    private final PostalCodeRepository repository;
    private final MutableLiveData<String> addressInput = new MutableLiveData();
    public final LiveData<String> postalCode =
            Transformations.switchMap(addressInput, (address) -> {
                return repository.getPostCode(address);
             });

  public MyViewModel(PostalCodeRepository repository) {
      this.repository = repository
  }

  private void setInput(String address) {
      addressInput.setValue(address);
  }
}
```

在上面的例子中，`postalCode` 字段永不变化，因而是 `public` 而 `final` 的；`postalCode` 字段被定义为 `addressInput` 的变换，也就意味着 `repository.getPostCode()` 方法会在 `addressInput` 改变时被调用。当然，这只是有观测者的情况，如果在 `repository.getPostCode()` 被调用时没有活动状态的观测者，那么在添加观测者之前，计算都不会被执行。

这种机制允许应用的低级部分创建惰性求值的 `LiveData` 对象。一个 `ViewModel` 对象能轻易获得对 `LiveData` 对象的引用，然后在此基础上定义对其的变换。

### 创建新的变换

您的应用可能需要一大堆各不相同的变换，但它们并不是默认提供的。想要实现自己的变换，您可以使用 [`MediatorLiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/MediatorLiveData.html) 类，该类监听其他 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象并处理它们发出的事件。`MediatorLiveData` 正确地将其状态传递给来源的 `LiveData` 对象。欲了解更多有关该模式的内容，请参阅 [`Transformations`](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html) 的文档。

## 合并多个 LiveData 数据源

[`MediatorLiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/MediatorLiveData.html) 是 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 的一个子类，允许您合并多个 LiveData 数据源。当任何 LiveData 数据源对象发生改变时， `MediatorLiveData` 对象的观测者都会被通知。

例如，如果您在 UI 中有一个可以被本地数据库或网络更新的 `LiveData` 对象，那么您可以将下列数据源添加到 `MediatorLiveData` 对象：

* 一个与数据库中的数据关联的 `LiveData` 对象
* 一个与从网络获取的数据关联的 `LiveData` 对象

您的 activity 只需观测 `MediatorLiveData` 对象来接收两个数据源的更新。欲了解更详尽的例子，请参阅[应用架构指南](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/B_Get_started/2_Guide_to_app_architecture.md)中的“附录：显示网络状态”。

## 其他资源

[`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 是 [Android Jetpack](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/A_Overview.md) 的一个架构组件。[Sunflower](https://github.com/googlesamples/android-sunflower) 范例应用展示了如何使用它们。

欲了解更多有关和 [`Snackbar`](https://developer.android.google.cn/training/snackbar) 信息、导航事件等一同使用 `LiveData` 的内容，请参阅[该博客](https://medium.com/google-developers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150)。

