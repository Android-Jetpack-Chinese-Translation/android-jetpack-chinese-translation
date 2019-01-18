# 双向数据绑定
> 原文链接：[Two-way data binding  |  Android Developers](https://developer.android.google.cn/topic/libraries/data-binding/two-way)

使用单向数据绑定，您可以为一个属性赋值，并设置一个监听器来针对该属性的数据变动做出反应。

```xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{viewmodel.rememberMeChanged}"
/>
```

而双向数据绑定则为这种流程提供了一种捷径。

```xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@={viewmodel.rememberMe}"
/>
```

特意包含了 `=` 的 `@={}` 记号会接收该属性的数据变动，并同时监听用户做出的更新。

为了针对其数据做出反应，您可以让布局变量实现 `Observable` 接口，一般就用 [`BaseObservable`](https://developer.android.com/reference/android/databinding/BaseObservable?hl=zh-cn)，并添加 [`@Bindable`](https://developer.android.com/reference/android/databinding/Bindable?hl=zh-cn) 注解，如下所示：

```java
public class LoginViewModel extends BaseObservable {
    // private Model data = ...

    @Bindable
    public Boolean getRememberMe() {
        return data.rememberMe;
    }

    public void setRememberMe(Boolean value) {
        // 避免无限循环。
        if (data.rememberMe != value) {
            data.rememberMe = value;

            // 针对变动做出反应。
            saveData();

            // 把新的值通知给观测者。
            notifyPropertyChanged(BR.remember_me);
        }
    }
}
```

由于可绑定的属性的 getter 叫做 `getRemeberMe()`，该属性对应的 setter 会被自动命名为 `setRememberMe()`。

欲了解更多使用 `BaseObservable` 和 `@Bindable` 的信息，请参阅 [与可观测的数据对象一起使用](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_4_Work_with_observable_data_objects.md)。

## 使用自定义的属性来双向数据绑定

平台为“最常见的双向属性”和变动监听器提供了双向数据绑定的实现，供您用在应用中。如果您想要使用自定义的属性来双向数据绑定，那您就需要使用 [`@InverseBindingAdapter`](https://developer.android.com/reference/android/databinding/InverseBindingAdapter?hl=zh-cn) 和 [`@InverseBindingMethod`](https://developer.android.com/reference/android/databinding/InverseBindingMethod?hl=zh-cn)。

例如，如果您想要在一个叫做 `MyView` 的自定义视图的 `"time"` 属性上启用双向数据绑定，请完成以下步骤：

1. 把 `@BindingAdapter` 注解添加到设置初始值并在数据变动时更新的方法上：

	```java
	@BindingAdapter("time")
	public static void setTime(MyView view, Time newValue) {
	    // 打破无限循环是很重要的！
	    if (view.time != newValue) {
	        view.time = newValue;
	    }
	}
	```

2. 把 `@InverseBindingAdapter` 注解添加到从视图读取数据的方法上：

	```java
	@InverseBindingAdapter("time")
	public static Time getTime(MyView view) {
	    return view.getTime();
	}
	```
	
在这个节点上，数据绑定已经知道当数据变动时应该调用注释有 `@BindingAdapter` 的方法、当视图属性变动时应该调用注释有 `@InverseBindingAdapter` 的方法了。然而，它还不知道属性会在什么时候、以什么方式变化。

为了解决这个问题，您需要在视图上设置一个监听器：它可以是一个和您自定义的视图相关联的自定义监听器，或者一个像失去焦点或者文本变动的通用事件。把 `@BindingAdapter` 注解添加到为属性的数据变动设置监听器的方法上，如下所示：

```java
@BindingAdapter("app:timeAttrChanged")
public static void setListeners(
        MyView view, final InverseBindingListener attrChange) {
    // 设置一个点击、焦点、触摸等操作的监听器。
}
```

监听器包含了一个 `InverseBindingListener` 作为参数，您可以用它来通知数据绑定系统关于数据已经变动的事实，然后系统就会开始调用注解了 `@InverseBindingAdapter` 的方法，以此类推。

> **注意**：每个双向绑定都会生成一个**合成的事件属性**。该属性和基类属性有着相同的名字，但包含了 `AttrChanged` 的后缀。该属性让数据绑定库能够创建一个用 `@BindingAdapter` 注解的方法，用于把事件监听器联系到合适的 `View` 实例上。

在实践中，监听器包含了一些并不琐碎的逻辑，例如单向数据绑定的监听器。欲获取一个例子，请参阅文本属性变化的适配器：[`TextViewBindingAdapter`](https://android.googlesource.com/platform/frameworks/data-binding/+/3b920788e90bb0abe615a5d5c899915f0014444b/extensions/baseAdapters/src/main/java/android/databinding/adapters/TextViewBindingAdapter.java#344)。

## 转换器（Converter）

如果和 `View` 对象绑定的变量需要在展示前被格式化、翻译或进行某种改变，您可以使用 `Converter` 对象。

下面的例子是了一个展示时间的 `EditText` 对象：

```xml
<EditText
    android:id="@+id/birth_date"
    android:text="@={Converter.dateToString(viewmodel.birthDate)}"
/>
```

`viewmodel.birthDate` 属性包含了一个 `Long` 类型的值，因此它需要使用一个转换器进行转换。

由于它使用了双向表达式，我们还需要一个*逆向转换器*来让数据绑定库明白如何把用户提供的字符串转换回它本身的数据类型，在该例中即 `Long`。这个过程需要为它的转换器之一添加 [`@InverseMethod`](https://developer.android.com/reference/android/databinding/InverseMethod?hl=zh-cn) 注解，并让该注解引用其逆向转换器。下面的例子展示了这种配置：

```java
public class Converter {
    @InverseMethod("stringToDate")
    public static String dateToString(EditText view, long oldValue,
            long value) {
        // 把 long 转换成 String。
    }

    public static long stringToDate(EditText view, String oldValue,
            String value) {
        // 把 String 转换成 long。
    }
}
```

## 使用双向绑定导致的无限循环

在使用双向绑定的时候，千万小心不要引入无限循环。当用户改变一个属性时，使用 `@InverseBindingAdapter` 注解的方法被调用，来把值赋给相应的属性；相应地，这又会调用使用 `@BindingAdapter` 注解的方法，于是再次调用 `@InverseBindingAdapter` 注解的方法，周而复始。

由于这个原因，一定要在 `@BindingAdapter` 注解的方法中，通过比较新值和旧值的办法来打破可能的无限循环。


## 双向属性

当您使用下表提供的属性时，平台已经提供了对双向绑定的内置支持。欲了解更多有关平台如何提供这种支持的信息，请参阅相应的绑定适配器：

| 类 | 属性 | 绑定适配器 |
| --- | --- | --- |
| AdapterView | android:selectedItemPosition android:selection | AdapterViewBindingAdapter |
| CalendarView | android:date | CalendarViewBindingAdapter |
| CompoundButton | android:checked | CompoundButtonBindingAdapter |
| DatePicker | android:year android:month android:day | DatePickerBindingAdapter |
| NumberPicker | android:value | NumberPickerBindingAdapter |
| RadioButton | android:checkedButton | RadioGroupBindingAdapter |
| RatingBar | android:rating | RatingBarBindingAdapter |
| SeekBar | android:progress | SeekBarBindingAdapter |
| TabHost | android:currentTab | TabHostBindingAdapter |
| TextView | android:text | TextViewBindingAdapter |
| TimePicker | android:hour android:minute | TimePickerBindingAdapter |

## 其他资源

GitHub 上的 [TwoWaySample](https://github.com/googlesamples/android-databinding/tree/master/TwoWaySample) 范例从头到尾地展示了本节提及的概念。


