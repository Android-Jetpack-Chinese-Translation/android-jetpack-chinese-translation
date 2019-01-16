# 自动生成的绑定类
> 原文链接：[Generated binding classes  |  Android Developers](https://developer.android.google.cn/topic/libraries/data-binding/generated-binding)

数据绑定库能自动生成绑定类，用来访问布局的变量和视图。本节展示了如何创建和定制这些自动生成的绑定类。

生成的绑定类把布局的变量和视图联系起来。绑定类的名字和包结构可以被定制。所有自动生成的绑定类都继承了 [`ViewDataBinding`](https://developer.android.com/reference/android/databinding/ViewDataBinding.html?hl=zh-cn) 类。

每个布局文件都能自动生成一个绑定类。默认情况下，绑定类的名字基于布局文件的文件名，将其转换成 Pascal 命名法，并添加 *binding* 后缀。上述布局文件的文件名是 `activity_main.xml`，因此其对应的绑定类就叫 `ActivityMainBinding`。该类持有其布局中属性（如 `user` 变量）到视图的所有绑定关系，并知道如何为绑定表达式赋值。

## 创建一个绑定对象

绑定对象应当在布局填充（inflate）完毕后尽快被创建，以免视图层次结构在绑定到布局内含有表达式的视图之前被修改。绑定对象到布局的最常用手段是调用绑定类的静态方法。您可以使用绑定类的 `infalte()` 方法来 填充视图结构并将其绑定到对象，如下例所示：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    MyLayoutBinding binding = MyLayoutBinding.inflate(getLayoutInflater());
}
```

`inflate()` 方法还有一个在 [`LayoutInflater`](https://developer.android.com/reference/android/view/LayoutInflater.html?hl=zh-cn) 对象之外还接收一个 [`ViewGroup`](https://developer.android.com/reference/android/view/ViewGroup.html?hl=zh-cn) 对象的变体，如下例所示：

```java
MyLayoutBinding binding = MyLayoutBinding.inflate(getLayoutInflater(), viewGroup, false);
```

如果布局是使用另一种机制被填充的话，它就能被单独绑定，如下所示：

```java
MyLayoutBinding binding = MyLayoutBinding.bind(viewRoot);
```

有时绑定的类型并不能被提前得知。在这种情况下，绑定类可以用 [`DataBindingUtil`](https://developer.android.com/reference/android/databinding/DataBindingUtil?hl=zh-cn) 类创建，如下例所示：

```java
View viewRoot = LayoutInflater.from(this).inflate(layoutId, parent, attachToParent);
ViewDataBinding binding = DataBindingUtil.bind(viewRoot);
```

如果您在 [`Fragment`](https://developer.android.com/reference/android/app/Fragment.html?hl=zh-cn)、[`ListView`](https://developer.android.com/reference/android/widget/ListView.html?hl=zh-cn) 或 [`RecyclerView`](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html?hl=zh-cn) 适配器中使用数据绑定的项目，您也许会更想使用绑定类的 [`inflate()`](https://developer.android.com/reference/android/databinding/DataBindingUtil.html?hl=zh-cn#inflate(android.view.LayoutInflater,%20int,%20android.view.ViewGroup,%20boolean,%20android.databinding.DataBindingComponent)) 方法或者 [`DataBindingUtil`](https://developer.android.com/reference/android/databinding/DataBindingUtil?hl=zh-cn)，如下例所示：

```java
ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
// 或者
ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
```

## 含有 ID 的视图

数据绑定库会在绑定类中为布局内的每个含有 ID 的视图都创建一个不可修改的字段。例如，数据绑定库为下列布局创建两个 `TextView` 类型的字段：`firstName` 和 `lastName`。

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
   android:id="@+id/firstName"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"
  android:id="@+id/lastName"/>
   </LinearLayout>
</layout>
```

数据绑定库通过单次扫描，从视图层次中抽取出所有含有 ID 的视图。相比于在每个视图上都调用 [`findViewById()`](https://developer.android.com/reference/android/app/Activity.html?hl=zh-cn#findViewById(int)) 方法而言，这种机制要快得多。

在数据绑定中 ID 并不是那么重要，但仍存在一些必须从代码中访问视图的情形。

## 变量 

数据绑定库为布局中声明的每个变量都生成访问器。例如，下面的布局会在绑定类中为 `user`、`image` 和 `note` 变量生成 setter 和 getter。

```xml
<data>
   <import type="android.graphics.drawable.Drawable"/>
   <variable name="user" type="com.example.User"/>
   <variable name="image" type="Drawable"/>
   <variable name="note" type="String"/>
</data>
```

## ViewStub

[`ViewStub`](https://developer.android.com/reference/android/view/ViewStub.html?hl=zh-cn) 对象和普通的视图不同，它们一开始是不可见的。当它们被设置成可见的、或被显式要求填充时，它们就会填充另一个布局，并用其替换掉自己。

由于 `ViewStub` 终究要从视图层次结构中消失，绑定对象中的视图也必须消失以便被垃圾回收。由于视图是 `final` 属性的，一个 [`ViewStubProxy`](https://developer.android.com/reference/android/databinding/ViewStubProxy.html?hl=zh-cn) 对象就会取代 `ViewStub` 在自动生成的绑定类中的位置，让您能在 `ViewStub` 存在的时候能够访问到它，并在 `ViewStub` 被填充后能够访问到被填充的视图结构。

当填充上述的另一个布局时，新的布局必须建立一个绑定关系。因此，`ViewStubProxy` 必须监听 `ViewStub` 的 [`OnInflateListener`](https://developer.android.com/reference/android/view/ViewStub.OnInflateListener.html?hl=zh-cn)，并在被要求的时候建立绑定。由于任何时候都只允许一个监听器存在，`ViewStubProxy` 允许您设置一个 `OnInflateListener`，并在绑定被建立时调用它。

## 立即绑定

当一个变量或可观测的对象发生变化时，绑定会被安排在下一帧开始前更新。然而，也有绑定必须被立即执行的情况。欲强制执行绑定，请调用 [`executePendingBindings()`](https://developer.android.com/reference/android/databinding/ViewDataBinding.html?hl=zh-cn#executePendingBindings())。

## 高级绑定

### 动态变量

有时，特定绑定类的种类无从得知。例如，一个处理任意布局的 [`RecyclerView.Adapter`](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html?hl=zh-cn) 并不知道其绑定类，然而它仍必须在 [`onBindViewHolder`](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html?hl=zh-cn#onBindViewHolder(VH,%20int)) 被调用时给绑定赋值。

如下所示，`RecyclerView` 绑定的所有布局都有一个 `item` 变量。`BindingHolder` 对象有一个 `getBinding()` 方法，用来返回 [`ViewDataBinding`](https://developer.android.com/reference/android/databinding/ViewDataBinding.html?hl=zh-cn) 的基类。

```java
public void onBindViewHolder(BindingHolder holder, int position) {
    final T item = mItems.get(position);
    holder.getBinding().setVariable(BR.item, item);
    holder.getBinding().executePendingBindings();
}
```

> **注意**：数据绑定库会在含有用于数据绑定的资源ID的模块包里生成一个叫 **BR** 的类。在上例中，数据绑定库自动生成了 **BR.item** 变量。


## 后台线程

只要您的数据模型不是一个集合，您就能在一个后台线程中修改它。数据绑定在计算阶段会将每个变量/字段本地化，以免出现任何并发问题。

## 定制绑定类名

默认情况下，绑定类的名字基于布局文件的文件名，将首字母大写，移除下划线，再将接下来的一个字母大写，并添加 *binding* 后缀。该类会被置于模块的一个 `dataBinding` 包中。例如，一个名为 `contact_item.xml` 的布局文件，其对应的绑定类就叫 `ContactItemBinding`。如果模块的包是 `com.example.my.app`，那么该绑定类就会被置于 `com.example.my.app.databinding` 中。

绑定类可以通过调整 `data` 元素的 `class` 属性的方法，被重命名或者移动到其他包中。例如，下列布局文件会在当前模块的 `databinding` 包中生成 `ContactItem` 绑定：

```xml
<data class="ContactItem">
    …
</data>
```

您可以通过在类名前增加一个句号的方法，使其生成到另一个包中。下列布局文件会在当前模块中生成 `ContactItem` 绑定：

```xml
<data class=".ContactItem">
    …
</data>
```

您也可以标明完整的包名，来让绑定类生成到该包的位置。下列布局文件会在`com.example` 包中生成 `ContactItem` 绑定：

```xml
<data class="com.example.ContactItem">
    …
</data>
```

