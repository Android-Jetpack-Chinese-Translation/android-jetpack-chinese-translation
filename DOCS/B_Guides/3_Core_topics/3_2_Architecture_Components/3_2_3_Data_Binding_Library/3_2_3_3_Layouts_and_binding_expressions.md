# 布局文件和绑定表达式
> 原文链接：[Layouts and binding expressions  |  Android Developers](https://developer.android.google.cn/topic/libraries/data-binding/expressions)

您可以使用数据绑定的表达式语言来编写出能够处理视图分发的事件的表达式。数据绑定自动地生成所需要的文件，用来把布局文件中的视图绑定到您的数据对象上。

数据绑定的布局文件有一点特别：它们开始于一个 `layout` 的的根节点，接着是一个 `data` 和一个 `view` 的节点，该 `view` 节点就是您会在普通的布局文件中使用的根节点。如下的代码展示了一个布局文件的例子：

```xml
<?xml version="1.0" encoding="utf-8"?>
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
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

`data` 节点中的 `user` 变量名描述了一个可以用于该布局文件的属性。

```xml
<variable name="user" type="com.example.User" />
```

布局文件中的表达式使用`@{}`的格式来表示属性。在这个例子中，`TextView`的文本被设置成 `user` 变量的 `firstName` 属性：

```xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}" />
```

> **注意**：布局表达式应当保持精简，因为它们不能被单元测试，也只有受限制的 IDE 支持。为了简化布局表达式，您可以使用定制的 [绑定适配器](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_6_Binding_adapters.md)。


## 数据对象

假设您现在有一个老旧的对象来描述 `User` 实体：

```java
public class User {
  public final String firstName;
  public final String lastName;
  public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
  }
}
```

这种对象含有永不改变的数据。这种做法很常见：在应用中使用只被读取一次后就再也不变更的数据。也有可能使用符合一系列规范的对象，例如 Java 中的 getter 方法，如下例所示：

```java
public class User {
  private final String firstName;
  private final String lastName;
  public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
  }
  public String getFirstName() {
      return this.firstName;
  }
  public String getLastName() {
      return this.lastName;
  }
}
```

从数据绑定的角度说，这两个类是等价的。用于 `android:text` 属性的 `@{user.firstName}` 表达式使用后者的 `getFirstName()` 方法来获取前者的 `firstName` 属性。或者，如果 `firstName()` 存在的话，它也会使用。

## 数据绑定

每个布局文件都会生成一个绑定类。默认情况下，该类的类名是基于布局文件的文件名的：将布局文件名转换成 Pascal 命名法，并向其添加 `Binding` 后缀。上例中的布局文件是 `activity_main.xml`，因此自动生成的对应的绑定类就是 `ActivityMainBinding`。该类持有从布局的属性（例如 `user` 变量）指向布局的视图的所有绑定，并指导如何向绑定表达式赋值。建议在填充布局文件时创建其绑定，如下例所示：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```
在运行时，应用会在 UI 中显示 **Test** 用户。或者，您可以使用 [`LayoutInflater`](https://developer.android.com/reference/android/view/LayoutInflater.html?hl=zh-cn) 来获取其视图，如下例所示：

```java
ActivityMainBinding binding = ActivityMainBinding.inflate(getLayoutInflater());
```

如果您在[`Fragment`](https://developer.android.com/reference/android/app/Fragment.html?hl=zh-cn)、[`ListView`](https://developer.android.com/reference/android/widget/ListView.html?hl=zh-cn)或[`RecyclerView`](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html?hl=zh-cn)的适配器中使用数据绑定的项目，您或许会使用绑定类的 [`inflate()`](https://developer.android.com/reference/android/databinding/DataBindingUtil.html?hl=zh-cn#inflate(android.view.LayoutInflater,%20int,%20android.view.ViewGroup,%20boolean,%20android.databinding.DataBindingComponent)) 方法、或者 [`DataBindingUtil`](https://developer.android.com/reference/android/databinding/DataBindingUtil?hl=zh-cn) 类，如下例所示：

```java
ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
// or
ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
```

## 表达式语言

### 常见特性

表达式语言看起来很像是托管代码。您可以在表达式语言中使用如下的运算符和关键词：

* 数学：+ - / * %
* 字符串连接：+
* 逻辑：&& ||
* 二进制：& | ^
* 一元：+ - ! ~
* 移位：>> >>> <<
* 比较：== > < >= <=
* `instanceof`
* 组合：`()`
* 字面量（Literals）：字符、字符串、数字、null
* 类转换
* 方法调用
* 字段访问
* 数组访问 `[]`
* 三元：`?:`

例子：

```xml
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```
### 缺失的运算

表达式语言中缺失了托管代码中可以使用的如下运算：

* `this`
* `super`
* `new`
* 显式的泛型

### Null 合并运算符（coalescing operator）

当左边的运算数不是 `null` 时，Null 合并运算符（`??`）选择左边；否则，选择右边。

```xml
android:text="@{user.displayName ?? user.lastName}"
```

这在功能上等价于：

```xml
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

### 属性引用

一个表达式可以使用如下的格式来引用一个类中的属性、字段、getter 和 [`ObservableField`](https://developer.android.com/reference/android/databinding/ObservableField.html?hl=zh-cn) 对象：

```xml
android:text="@{user.lastName}"
```

### 避免空指针异常

生成的数据绑定代码能自动检查 `null` 值并避免空指针异常。例如，在 `@{user.name}`，如果 `user` 是空值，那么 `user.name` 会被赋予默认值 `null`。如果你引用 `user.age`（其中 `age` 是 `int` 类型），那么数据绑定就会使用其默认值 `0`。

### 集合

数组、列表、稀疏列表（sparse list）和键值映射表（map）等常见的集合都可以使用 `[]` 运算符来便利地访问。

```xml
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List<String>"/>
    <variable name="sparse" type="SparseArray<String>"/>
    <variable name="map" type="Map<String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
```

> **注意**：您也可以使用 `object.key` 来引用一个表中的值。例如，上述例子中的 `@{map[key]}` 可以用 `@{map.key}` 来取代。


### 字符串字面量

您可以使用单括号来包围属性值，这样您就能在表达式中使用双括号，如下例所示：

```xml
android:text='@{map["firstName"]}'
```

您也可以使用双括号来包围属性值。这样做的时候，字符串字面量应当被  `` ` ``包围：

```xml
android:text="@{map[`firstName`]}"
```

### 资源

您可以在表达式中使用如下的语法来访问资源：

```xml
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```

格式化字符串和复数可以使用如下的参数来计算：

```xml
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```

当一个复数接收多个参数时，所有的参数都应被传入：

```xml
  Have an orange
  Have %d oranges

android:text="@{@plurals/orange(orangeCount, orangeCount)}"
```

一些资源要求显示的类计算，如下表所示：

| 类 | 普通的引用 | 表达式引用 | 
|---|---|---|
| String[] | @array | @stringArray |
| int[] | @array | @intArray |
| TypedArray | @array | @typedArray |
| Animator | @animator | @animator |
| StateListAnimator | @animator | @stateListAnimator |
| color | int | @color | @color |
| ColorStateList | @color | @colorStateList |

### 事件处理

您可以使用数据绑定的表达式语言来编写出能够处理视图分发的事件（例如，[`onClick()`](https://developer.android.com/reference/android/view/View.OnClickListener.html?hl=zh-cn#onClick(android.view.View))）的表达式。事件的属性名是由监听器方法决定的，但也有一些例外。例如，[`View.OnClickListener`](https://developer.android.com/reference/android/view/View.OnClickListener.html?hl=zh-cn) 有一个 [`onClick()`](https://developer.android.com/reference/android/view/View.OnClickListener.html?hl=zh-cn#onClick(android.view.View))）方法，因此该事件的属性就是 `android:onClick`。

点击事件有一些特殊的事件处理器，它们需要 `android:onClick` 以外的属性来避免冲突。您可以使用如下的属性来避免这种冲突：

| 类 | 监听器 setter | 属性 |
| --- | --- | --- |
| SearchView | setOnSearchClickListener(View.OnClickListener) | android:onSearchClick |
| ZoomControls | setOnZoomInClickListener(View.OnClickListener) | android:onZoomIn |
| ZoomControls | setOnZoomOutClickListener(View.OnClickListener) | android:onZoomOut |

您可以利用如下的机制来处理事件：

* 方法引用：在您的表达式中，您可以引用符合监听器方法的方法签名（method signature）的方法。当一个表达式计算结果是一个方法的引用时，数据绑定会将方法的引用和拥有它的对象包裹在一个监听器中，并将该监听器设置到目标视图上；如果该表达式的计算结果是 `null`，那么数据绑定就只在目标视图上设置一个 `null` 监听器。
* 监听器绑定：在事件发生时将会被计算结果的 lambda 表达式。数据绑定总是创建一个监听器来设置到视图上。当事件被分发时，这些监听器就会计算这些 lambda 表达式。

### 方法引用

事件可以被直接限定到处理器的方法上，就像 `android:onClick` 可以被赋予 activity 中的一个办法。和[`View`](https://developer.android.com/reference/android/view/View.html?hl=zh-cn) `onClick` 属性想必，一个主要的优势是：表达式会在编译时被处理，因此如果该方法不存在或者其方法签名不正确，您就会收到一个编译时错误。

方法引用和监听器绑定的一个主要差异是：实际的监听器实现是在数据被限定时创建的，而非事件被触发时。如果您更喜欢在事件发生时计算表达式结果，您应该使用监听器绑定。

欲把一个事件赋给其处理器，请使用一个值为欲调用的方法名的普通的绑定表达式。例如，考虑如下的布局数据对象例子：

```java
public class MyHandlers {
    public void onClickFriend(View view) { ... }
}
```

绑定表达式可以将一个视图的点击监听器赋值给 `onClickFriend()` 方法，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.MyHandlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```

> **注意**：表达式中的方法签名必须完全符合监听器对象中的方法。

### 监听器绑定

监听器绑定指的是事件发生时才运行的绑定表达式。它们和方法引用很像，但允许您运行任意的数据绑定表达式。这个特性在适配于 Gradle 2.0 版本以上的 Android Gradle 插件才支持。

在方法引用的场景中，方法的参数必须和事件监听器的方法完全一致；而在监听器绑定的情况下，只有您的返回值必须和监听器期待的返回值保持一致，除非它本来就期待的是空值。例如，考虑如下的有 `onSaveClick()` 的 presenter 类：

```java
public class Presenter {
    public void onSaveClick(Task task){}
}
```

那么您就可以把点击事件绑定到 `onSaveClick()`，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="task" type="com.android.example.Task" />
        <variable name="presenter" type="com.android.example.Presenter" />
    </data>
    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
        <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onClick="@{() -> presenter.onSaveClick(task)}" />
    </LinearLayout>
</layout>
```

当一个回调被用于表达式中，数据绑定能自动创建所需要的监听器，并将其注册到该回调事件上。当视图触发事件时，数据绑定就会计算该表达式。就像普通的绑定表达式一样，在它们被计算的时候，您仍能得到数据绑定的空值安全和线程安全特性。

在上例中，我们还没有定义被传入 [`onClick(View)`](https://developer.android.com/reference/android/view/View.OnClickListener.html?hl=zh-cn#onClick(android.view.View)) 的 `view` 参数。监听器绑定提供了两种监听器参数：您要么省略所有参数，要么指定所有参数。后者的话，您可以在表达式中使用它们。例如，上述的表达式可以写成下面的样子：

```xml
android:onClick="@{(view) -> presenter.onSaveClick(task)}"
```

或者，如果您想要在表达式中使用参数，也可以这样写：

```java
public class Presenter {
    public void onSaveClick(View view, Task task){}
}
```
```xml
android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```

您可以使用一个含有多个参数的 lambda 表达式：

```java
public class Presenter {
    public void onCompletedChanged(Task task, boolean completed){}
}
```

```xml
<CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
      android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```

如果您正在监听的事件返回了一个不是 `void` 的值，您的表达式也必须返回相同的值。例如，如果您想要监听长点击事件，那么您的表达式就应该返回一个 boolean。

```java
public class Presenter {
    public boolean onLongClick(View view, Task task) { }
}
```
```xml
android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
```

如果表达式因 `null` 对象的原因不能被计算，数据绑定就会返回该类的默认值。例如，引用类型的 `null`，`int` 的 `0`，`boolean` 的 `false`。

如果您需要在表达式中使用一个谓词（predicate），例如三元表达式，您您可以将 `void` 用作一个符号：

```xml
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

### 避免复杂的监听器

监听器表达式非常强大，并能让您的代码容易阅读；而另一方面，在监听器中使用复杂的表达式则会导致您的布局文件难以阅读和维护。这些表达式应当精简到只用于将可用的数据从UI传到您的回调方法。您应将业务逻辑实现在监听器表达式所触发的回调方法中。

## 导入、变量、包含布局

数据绑定库提供了导入、变量和包含的功能。导入（import）让您在布局文件中轻松地引用所需的类；变量（variables）允许您描述一个能用于绑定表达式中的属性；包含布局（includes）让您能在应用中复用复杂的布局。

### 导入

导入让您在布局文件中轻松地引用所需的类，就像托管代码一样。`data` 元素中可以使用零或多个 `import` 元素。下例将 `View` 类导入到您的布局文件：

```xml
<data>
    <import type="android.view.View"/>
</data>
```

导入 `View` 类允许您从绑定表达式中引用它。下例展示了如何引用 `View` 类中的 `VISIBLE` 和 `GONE` 常量：

```xml
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

### 类型别名（aliases）

当类名发生冲突时，其中一个可以被重命名成一个别名。下例将 `com.example.real.estate` 包中的 `View` 重命名成 `Vista` ：

```xml
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

### 导入其他类

导入的类型可被变量和表达式引用。下例展示了 `User` 和 `List` 被用于一个变量的类型：

```xml
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List<User>"/>
</data>
```

> **警告**：Android Studio 还不能处理该导入功能，因此您 IDE 为导入的自动补全功能可能并不工作。您的应用仍能编译，您也可以在变量的定义中使用完全的类名来绕开该问题。


您也可以用导入的类型来转换部分表达式。下例将 `connection` 属性转换成了 `User` 类：

```xml
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

导入的类型也能也能用于在表达式中引用静态字段和方法。下例导入了 `MyStringUtils` 类，并引用了它的 `capitalize` 方法：

```xml
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

就像托管代码一样，`java.lang.*` 会被自动导入。

### 变量

您可以在 `data` 元素中使用多个 `variable` 元素，每个 `variable` 元素都描述了一个可以被设置到布局上的属性，用于该布局文件中的绑定表达式。下例声明了 `user`、`image` 和 `note` 变量：

```xml
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user" type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note" type="String"/>
</data>
```

变量的类型会在编译时被视察，因此如果一个变量实现了[`Observable`](https://developer.android.com/reference/android/databinding/Observable.html?hl=zh-cn)接口、或者是一个[可观测的集合](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_4_Work_with_observable_data_objects.md)，那么这就应该在类型中被反映出来。如果变量是一个基类或者一个没有实现 `Observable` 的接口，那么变量就不会被观测。

当处理不同配置（如横向屏幕和竖向屏幕）的多个布局文件存在时，其变量会被合并：这些布局文件必须不能含有冲突的变量定义。

自动生成的绑定类会为每个描述的变量提供一个 getter 和一个 setter。在 setter 被调用之前，这些变量取其托管代码类的默认值：引用类的 `null`，`int` 的 `0`，`boolean` 的 `false`，等等。

在绑定表达式需要的时候，一个叫做 `context` 的特殊变量会被生成，其值就是根视图调用[`getContext()`](https://developer.android.com/reference/android/view/View.html?hl=zh-cn#getContext())所得到的[`Context`](https://developer.android.com/reference/android/content/Context.html?hl=zh-cn)。`context` 变量会被一个同名的显式变量定义覆写。

### 包含布局

父布局能通过在 xml 属性中使用 app 命名空间和变量名的方式，将变量传入子布局的绑定。下例展示了 `name.xml` 和 `contact.xml` 包含的 `user` 变量：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```

数据绑定并不支持把包含布局直接作为一个 `<merge>` 的子元素。例如，*下例的布局文件是不被支持的*：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge><!-- 不好使 -->
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```


