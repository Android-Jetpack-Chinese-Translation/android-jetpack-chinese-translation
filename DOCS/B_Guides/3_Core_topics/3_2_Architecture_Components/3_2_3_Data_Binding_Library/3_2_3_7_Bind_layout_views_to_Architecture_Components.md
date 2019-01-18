# 将布局绑定到架构组件
> 原文链接：[Bind layout views to Architecture Components  |  Android Developers](https://developer.android.google.cn/topic/libraries/data-binding/architecture)

AndroidX 库包含了用于设计稳健、可测试且可维应用的[架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_1_Overview.md)。数据绑定库可以与这些组件无缝协作，用以进一步简化 UI 开发：这些架构组件已在帮助您管理 UI 控制器的生命周期、并在数据变化时自动通知，而您应用的布局又可以绑定到架构组件中的数据。

本节展示了如何将架构组件整合到您的应用中，以便更好地发挥数据绑定库的作用。

## 使用 LiveData 来把数据的变动通知给 UI

您可以把 [`LiveData`](https://developer.android.com/reference/android/arch/lifecycle/LiveData?hl=zh-cn) 对象用作数据绑定源来把数据的变动自动通知给 UI。欲了解更多关于架构组件的信息，请参阅 [LiveData 概览](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_5_LiveData.md)。

和 [可观测的字段](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_4_Work_with_observable_data_objects.md) 等实现了 [`Observable`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn) 接口的对象不同，`LiveData` 能感知到生命周期以及订阅了数据变动的观测者。这种感知能带来许多好处，如[使用 LiveData 的好处](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_5_LiveData.md)所述。在 3.1 及更高版本的 Android Studio 中，您可以用 `LiveData` 来取代数据绑定代码中的可观测的字段。

欲和您的数据绑定类一同使用 `LiveData` 对象，您需要指定一个生命周期所有者来定义 `LiveData` 的作用域。如下的代码在实例化绑定类之后，把 activity 指定为生命周期所有者：

```java
class ViewModelActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // 填充视图，获取绑定类的一个实例。
        UserBinding binding = DataBindingUtil.setContentView(this, R.layout.user);

        // 把当前的 activity 指定为生命周期所有者。
        binding.setLifecycleOwner(this);
    }
}
```

您可以使用一个 [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel?hl=zh-cn) 组件来把数据绑定到布局，如“使用 ViewModel 来管理与 UI 相关的数据”所述。在 `ViewModel` 组件中，您可以使用 `LiveData` 对象来转换数据或者合并多个数据源。如下的代码展示了如何在 `ViewModel` 中转换数据：

```java
class ScheduleViewModel extends ViewModel {
    LiveData username;

    public ScheduleViewModel() {
        String result = Repository.userName;
        userName = Transformations.map(result, result -> result.value);
    }
}
```

## 使用 ViewModel 来管理与 UI 相关的数据

数据绑定库能和 [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel?hl=zh-cn) 组件无缝协作：ViewModel 把数据暴露给进行观测的布局，而布局对数据的变化做出反应。和数据绑定库一起使用 `ViewModel` 组件让您能把 UI 逻辑从布局移到组件中，而后者更容易测试。数据绑定库保证视图在需要的时候能够和数据绑定/解绑，那么剩下的工作就主要是保证暴露的是正确的数据了。欲了解更多有关该架构组件的信息，请参阅 [ViewModel 概览](https://developer.android.com/topic/libraries/architecture/viewmodel.html?hl=zh-cn)。

欲和数据绑定库一同使用 `ViewModel` 组件，您必须初始化您的继承了 [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel.html?hl=zh-cn) 类的组件，获得一个您的绑定类的实例，并把您的 `ViewModel` 组件赋值到绑定类中的一个属性。如下例所示：

```java
class ViewModelActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // 获取 ViewModel 组件。
        UserModel userModel = ViewModelProviders.of(getActivity())
                                                  .get(UserModel.class);

        // 填充视图，获取绑定类的一个实例。
        UserBinding binding = DataBindingUtil.setContentView(this, R.layout.user);

        // 将组件赋值到绑定类中的一个属性。
        binding.viewmodel = userModel;
    }
}
```

在您的布局中，把您 `ViewModel` 组件中的属性和方法赋值给相应的使用绑定表达式的视图，如下例所示：

```xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{() -> viewmodel.rememberMeChanged()}" />
```

## 使用可观测的 ViewModel 来获取对绑定适配器的更多控制

您可以使用一个实现了 [`Observable`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn) 的 [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel?hl=zh-cn) 来把数据变动通知给其他应用组件，就像您使用 [`LiveData`](https://developer.android.com/reference/android/arch/lifecycle/LiveData?hl=zh-cn) 那样。

在一些情况下，相比于 `LiveData` 对象，您可能更想使用实现了 `Observable` 接口的 `ViewModel` 组件，即使这意味着会失去 `LiveData` 的生命周期管理能力。使用实现了 `Observable` 接口的 `ViewModel` 组件能让您获得对于应用中绑定适配器的更多控制。例如，这种机制让您能制定一个自定义的方法来通过双向绑定的方式设置一个属性的值。

欲实现一个可观测的 `ViewModel` 组件，您必须创建一个实现了 `Observabl` 的 `ViewModel` 子类。您可以在观测者调用  [`addOnPropertyChangedCallback()`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn#addOnPropertyChangedCallback(android.databinding.Observable.OnPropertyChangedCallback)) 和 [`removeOnPropertyChangedCallback()`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn#removeOnPropertyChangedCallback(android.databinding.Observable.OnPropertyChangedCallback)) 方法来订阅或取消订阅的时候提供您自定义的逻辑。下面的代码展示了如何实现一个可观测的 `ViewModel` 组件：

```java
/**
 * 一个可观测的 ViewModel，应和数据绑定库一起使用。
 */
class ObservableViewModel extends ViewModel implements Observable {
    private PropertyChangeRegistry callbacks = new PropertyChangeRegistry();

    @Override
    protected void addOnPropertyChangedCallback(
            Observable.OnPropertyChangedCallback callback) {
        callbacks.add(callback);
    }

    @Override
    protected void removeOnPropertyChangedCallback(
            Observable.OnPropertyChangedCallback callback) {
        callbacks.remove(callback);
    }

    /**
     * 把该实例中所有属性的数据变动通知给观测者。
     */
    void notifyChange() {
        callbacks.notifyCallbacks(this, 0, null);
    }

    /**
     * 把一个特定属性的数据变动通知观测者。
     * 该属性的 getter 应当被标记 @Bindable 注解，用以在 BR 类中生成一个可被用作 fieldID 参数的字段。
     *
     * @param fieldId：生成的 BR 中的绑定类的 id。
     */
    void notifyPropertyChanged(int fieldId) {
        callbacks.notifyCallbacks(this, fieldId, null);
    }
}
```

