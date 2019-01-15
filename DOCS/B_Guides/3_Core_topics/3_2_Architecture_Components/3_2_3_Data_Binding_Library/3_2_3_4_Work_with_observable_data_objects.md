# 与可观测的数据对象一起使用
> 原文链接：[Work with observable data objects  |  Android Developers](https://developer.android.google.cn/topic/libraries/data-binding/observability)

可观测性指的是一个对象能将自己数据的变化通知给其他对象（即：监听器）的能力。数据绑定库让您能够把对象、字段和集合变得可观测。

任何老旧的对象都能用于数据绑定，但是修改这些对象并不能让 UI 自动更新。数据绑定能赋予您的数据对象在数据变动时通知其监听器的能力。可观测的类有三种不同的类型：对象、字段和集合。

## 可观测的字段

创建一个实现了 [`Observable`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn) 接口的类可能会有些麻烦，如果您的类只有少数几个属性的话，这可能并不值得。在这种情况下，您可以使用通用的 [`Observable`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn) 类和下列针对原始类型的类来让字段变得可观测：

* [`ObservableBoolean`](https://developer.android.com/reference/android/databinding/ObservableBoolean.html?hl=zh-cn)
* [`ObservableByte`](https://developer.android.com/reference/android/databinding/ObservableByte.html?hl=zh-cn)
* [`ObservableChar`](https://developer.android.com/reference/android/databinding/ObservableChar.html?hl=zh-cn)
* [`ObservableShort`](https://developer.android.com/reference/android/databinding/ObservableShort.html?hl=zh-cn)
* [`ObservableInt`](https://developer.android.com/reference/android/databinding/ObservableInt.html?hl=zh-cn)
* [`ObservableLong`](https://developer.android.com/reference/android/databinding/ObservableLong.html?hl=zh-cn)
* [`ObservableFloat`](https://developer.android.com/reference/android/databinding/ObservableFloat.html?hl=zh-cn)
* [`ObservableDouble`](https://developer.android.com/reference/android/databinding/ObservableDouble.html?hl=zh-cn)
* [`ObservableParcelable`](https://developer.android.com/reference/android/databinding/ObservableParcelable.html?hl=zh-cn)

可观测的字段是自给自足的、仅含有一个字段的可观测对象。原始类型的版本避免了访问操作的装箱和拆箱。欲使用该机制，创建一个 Java 的 `public final` 的字段或者 Kotlin 的只读属性，如下例所示：

```java
private static class User {
    public final ObservableField<String> firstName = new ObservableField<>();
    public final ObservableField<String> lastName = new ObservableField<>();
    public final ObservableInt age = new ObservableInt();
}
```

```kotlin
class User {
    val firstName = ObservableField<String>()
    val lastName = ObservableField<String>()
    val age = ObservableInt()
}
```

欲访问这些字段的值，使用 [`set()`](https://developer.android.com/reference/android/databinding/ObservableField.html?hl=zh-cn#set) 和 [`get()`](https://developer.android.com/reference/android/databinding/ObservableField.html?hl=zh-cn#get) 方法，如下例所示：

```java
user.firstName.set("Google");
int age = user.age.get();
```

```kotlin
user.firstName = "Google"
val age = user.age
```

> **注意**：Android Studio 3.1 及更高版本允许您将可观测的字段替换为 LiveData 对象，这能给您的应用带来额外的好处。欲了解更多信息，请参阅：[使用 LiveData 把数据变动通知给 UI](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_7_Bind_layout_views_to_Architecture_Components.md)。

## 可观测的集合

不少应用使用动态结构来存储数据。可观测的集合允许使用一个键（key）来访问这些结构。当键是引用类型（如 `String`）时，[`ObservableArrayMap`](https://developer.android.com/reference/android/databinding/ObservableArrayMap.html?hl=zh-cn) 类就能派上用场，如下例所示：

```java
ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);
```

在布局文件中，该映射可以用 `String` 的键来访问，如下所示：

```xml
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>
…
<TextView
    android:text="@{user.lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text="@{String.valueOf(1 + (Integer)user.age)}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

当键是整型时，[`ObservableArrayList`](https://developer.android.com/reference/android/databinding/ObservableArrayList.html?hl=zh-cn) 类就能派上用场，如下例所示：

```java
ObservableArrayList<Object> user = new ObservableArrayList<>();
user.add("Google");
user.add("Inc.");
user.add(17);
```

在布局文件中，列表可以通过索引来访问，如下所示：

```xml
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList<Object>"/>
</data>
…
<TextView
    android:text='@{user[Fields.LAST_NAME]}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

## 可观测的对象

一个实现了 [`Observable`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn) 接口的类允许监听器注册，并让它们在属性的值变化时收到通知。

[`Observable`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn) 接口有一个添加和移除监听器的机制，但你能必须决定何时发送通知。为了简化开发流程，数据绑定库提供了 [`BaseObservable`](https://developer.android.com/reference/android/databinding/BaseObservable.html?hl=zh-cn) 类，该类已实现了监听器注册的机制。实现了 `BaseObservable` 的数据类需负责其属性变化的通知。这是通过给 getter 添加一个 [`Bindable`](https://developer.android.com/reference/android/databinding/Bindable.html?hl=zh-cn) 注解、并调用 setter 中的 [`notifyPropertyChanged()`](`https://developer.android.com/reference/android/databinding/BaseObservable.html?hl=zh-cn#notifyPropertyChanged(int)) 方法来实现的，如下例所示：

```java
private static class User extends BaseObservable {
    private String firstName;
    private String lastName;

    @Bindable
    public String getFirstName() {
        return this.firstName;
    }

    @Bindable
    public String getLastName() {
        return this.lastName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
        notifyPropertyChanged(BR.firstName);
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
        notifyPropertyChanged(BR.lastName);
    }
}
```

数据绑定在模块包里生成一个叫做 `BR` 的、包含了用于数据绑定的资源 ID 的类。[`Bindable`](https://developer.android.com/reference/android/databinding/Bindable.html?hl=zh-cn) 注解会在编译时向 `BR` 类生成一条项目。如果数据类的基类不能被改变，那么 [`Observable`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn) 接口就能通过一个 [`PropertyChangeRegistry`](https://developer.android.com/reference/android/databinding/PropertyChangeRegistry.html?hl=zh-cn) 对象来高效地注册并通知监听器。


