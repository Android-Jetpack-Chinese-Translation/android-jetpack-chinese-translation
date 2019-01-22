# 绑定适配器
> 原文链接：[Binding adapters  |  Android Developers](https://developer.android.google.cn/topic/libraries/data-binding/binding-adapters)

绑定适配器负责调用合适的框架 API 来赋值。例如，像 [`setText()`](https://developer.android.google.cn/reference/android/widget/TextView.html?hl=zh-cn#setText(char[],%20int,%20int)) 那样设置一个属性的值，以及 [`setOnClickListener()`](https://developer.android.google.cn/reference/android/view/View.html?hl=zh-cn#setOnClickListener(android.view.View.OnClickListener)) 那样设置一个事件监听器。

数据绑定库允许您指定赋值的方法，提供您自己的绑定逻辑，以及指定绑定适配器所返回的对象的类型。

## 设置属性值

无论一个绑定的值在何时发生变化，生成的绑定类都必须在含有绑定表达式的视图上调用一个 setter 方法。您可以让数据绑定库自动选择这个方法，或者显式地声明一个方法，甚至直接提供一个选择方法的逻辑。

### 自动选择方法

以一个名为 `example` 的属性而言，数据绑定库会自动尝试搜索一个名为 `setExample(arg)` 的、接受一个兼容的参数类型的方法，这种搜索只关心该属性的名字和类型，而忽略其命名空间。

例如，给定一个 `android:text="@{user.name}` 的表达式，数据绑定库搜索一个名为 `setText(arg)` 的、接收一个类型与 `user.getName()` 返回值相同的参数。如果表达式返回的一个 `int`，那么数据绑定库就会去搜索一个名为 `setText()` 但接收一个 `int` 参数的方法。表达式必须返回正确的类型，因而您在必要的时候可以把返回值显式转换类型。

即使没有找到相应名称的属性，数据绑定也能工作。这时您就可以使用数据绑定来为任何一个 setter 创建相应的属性。例如，支持库类 [`DrawerLayout`](https://developer.android.google.cn/reference/android/support/v4/widget/DrawerLayout.html?hl=zh-cn) 并没有任何属性，但有许多 setter。如下的布局自动选择[`setScrimColor(int)`](https://developer.android.google.cn/reference/android/support/v4/widget/DrawerLayout.html?hl=zh-cn#setScrimColor(int)) 和 [`setDrawerListener(DrawerListener)`](https://developer.android.google.cn/reference/android/support/v4/widget/DrawerLayout.html?hl=zh-cn#setDrawerListener(android.support.v4.widget.DrawerLayout.DrawerListener)) 方法作为 `app:scrimColor` 和 `app:drawerListener` 的 setter，如下所示：

```xml
<android.support.v4.widget.DrawerLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}">
```

### 指定自定义的方法名

一些属性有名字完全对不上号的 setter。这种情况下，一个属性可以通过 [`BindingMethods`](https://developer.android.google.cn/reference/android/databinding/BindingMethods.html?hl=zh-cn) 注解来和相应的 setter 联系起来。该注解是用在类上的，并能包括多个用于重命名方法的 [`BindingMethod`](https://developer.android.google.cn/reference/android/databinding/BindingMethod.html?hl=zh-cn) 注解。绑定方法是可以添加到您应用中任意类的注解。如下例所示，`android:tint` 属性是与 `setImageTintList(ColorStateList)` 方法联系起来的，而非 `setTint()`：

```java
@BindingMethods({
       @BindingMethod(type = "android.widget.ImageView",
                      attribute = "android:tint",
                      method = "setImageTintList"),
})
```

在大多数情况下，您并不需要重命名 Android 框架类中的 setter 方法。这些属性已经按照命名惯例实现，用于自动搜索相应的方法。

### 提供自定义的逻辑

一些属性需要自定义的绑定逻辑。例如，`android:paddingLeft` 属性并没有联系的 setter，然而 `setPadding(left, top, right, bottom)` 方法却是已提供的。一个 [`BindingAdapter`](https://developer.android.google.cn/reference/android/databinding/BindingAdapter.html?hl=zh-cn) 注释的静态的绑定适配器方法允许您自定义一个属性的 setter 被调用的方式。例如，下例展示了 `paddingLeft` 属性的绑定适配器：

```java
@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int padding) {
  view.setPadding(padding,
                  view.getPaddingTop(),
                  view.getPaddingRight(),
                  view.getPaddingBottom());
}
```

参数的类型是非常重要的。第一个参数决定了属性所对应的视图的种类，第二个参数决定了属性的绑定表达式所能接受的类型。

绑定适配器也有利于其他类型的自定义。例如，一个自定义的加载器可以从工作线程中被调用，用于加载一个图像。

当有冲突的时候，您定义的绑定适配器能够覆写掉 Android 框架默认的绑定适配器。

您也可以有接收多个属性的适配器，如下所示：

```java
@BindingAdapter({"imageUrl", "error"})
public static void loadImage(ImageView view, String url, Drawable error) {
  Picasso.get().load(url).error(error).into(view);
}
```

如下例所示，您可以在布局中使用适配器。注意 `@drawable/venueError` 指代的是您应用中的一个资源。使用 `@{}` 包围资源来使其成为有效的绑定表达式。

```xml
<ImageView app:imageUrl="@{venue.imageUrl}" app:error="@{@drawable/venueError}" />
```

> **注意**：数据绑定库在搜索适配器时会忽略自定义的命名空间。

如果 `imageUrl` 和 `error` 都被用于一个 `ImageView` 对象，而且前者是字符串而后者是 `Drawable` 的时候，适配器就会被调用。如果您想要适配器在*任何*属性被设置时都被调用，那么您可以把一个可选的 [`requireAll`](https://developer.android.google.cn/reference/android/databinding/BindingAdapter.html?hl=zh-cn#requireAll()) 标签参数（flag）设置为 `false`，如下例所示：

```java
@BindingAdapter(value={"imageUrl", "placeholder"}, requireAll=false)
public static void setImageUrl(ImageView imageView, String url, Drawable placeHolder) {
  if (url == null) {
    imageView.setImageDrawable(placeholder);
  } else {
    MyImageLoader.loadInto(imageView, url, placeholder);
  }
}
```

> **注意**：当有冲突的时候，您定义的绑定适配器能够覆写掉默认的绑定适配器。

绑定适配器的方法能在其处理器（handler）中可选地取旧值。一个能取旧值和新值的方法应当首先声明*所有*旧的值，接着才是新的值，如下例所示：

```java
@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int oldPadding, int newPadding) {
  if (oldPadding != newPadding) {
      view.setPadding(newPadding,
                      view.getPaddingTop(),
                      view.getPaddingRight(),
                      view.getPaddingBottom());
   }
}
```

事件处理器只能被用于接口或只有一个抽象方法的抽象类，如下例所示：

```java
@BindingAdapter("android:onLayoutChange")
public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue,
       View.OnLayoutChangeListener newValue) {
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
    if (oldValue != null) {
      view.removeOnLayoutChangeListener(oldValue);
    }
    if (newValue != null) {
      view.addOnLayoutChangeListener(newValue);
    }
  }
}
```

如下所示，在您的布局中使用这样的事件处理器：

```xml
<View android:onLayoutChange="@{() -> handler.layoutChanged()}"/>
```

当一个监听器含有多个方法时，它必须被分成多个监听器。例如，[`View.OnAttachStateChangeListener`](https://developer.android.google.cn/reference/android/view/View.OnAttachStateChangeListener.html?hl=zh-cn) 有两个方法：[`onViewAttachedToWindow(View)`](https://developer.android.google.cn/reference/android/view/View.OnAttachStateChangeListener.html?hl=zh-cn#onViewAttachedToWindow(android.view.View)) 和 [`onViewDetachedFromWindow(View)`](https://developer.android.google.cn/reference/android/view/View.OnAttachStateChangeListener.html?hl=zh-cn#onViewDetachedFromWindow(android.view.View))。数据绑定库提供了两个接口来为其区分属性和处理器：

```java
@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewDetachedFromWindow {
  void onViewDetachedFromWindow(View v);
}

@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewAttachedToWindow {
  void onViewAttachedToWindow(View v);
}
```

由于改变一个监听器也会影响另一个，您需要一个和任何属性或两个属性都能工作的适配器。您可以把注解中的 [`requireAll`](https://developer.android.google.cn/reference/android/databinding/BindingAdapter.html?hl=zh-cn#requireAll()) 标签参数（flag）设置为 `false`，来说明不是每个属性都必须被赋一个绑定表达式的值，如下例所示：

```java
@BindingAdapter({"android:onViewDetachedFromWindow", "android:onViewAttachedToWindow"}, requireAll=false)
public static void setListener(View view, OnViewDetachedFromWindow detach, OnViewAttachedToWindow attach) {
    if (VERSION.SDK_INT >= VERSION_CODES.HONEYCOMB_MR1) {
        OnAttachStateChangeListener newListener;
        if (detach == null && attach == null) {
            newListener = null;
        } else {
            newListener = new OnAttachStateChangeListener() {
                @Override
                public void onViewAttachedToWindow(View v) {
                    if (attach != null) {
                        attach.onViewAttachedToWindow(v);
                    }
                }
                @Override
                public void onViewDetachedFromWindow(View v) {
                    if (detach != null) {
                        detach.onViewDetachedFromWindow(v);
                    }
                }
            };
        }

        OnAttachStateChangeListener oldListener = ListenerUtil.trackListener(view, newListener,
                R.id.onAttachStateChangeListener);
        if (oldListener != null) {
            view.removeOnAttachStateChangeListener(oldListener);
        }
        if (newListener != null) {
            view.addOnAttachStateChangeListener(newListener);
        }
    }
}
```

上面的例子只比普通的情况略微复杂一点，因为 [`View`](https://developer.android.google.cn/reference/android/view/View.html?hl=zh-cn) 类使用 [`addOnAttachStateChangeListener()`](https://developer.android.google.cn/reference/android/view/View.html?hl=zh-cn#addOnAttachStateChangeListener(android.view.View.OnAttachStateChangeListener)) 和 [`removeOnAttachStateChangeListener()`](https://developer.android.google.cn/reference/android/view/View.html?hl=zh-cn#removeOnAttachStateChangeListener(android.view.View.OnAttachStateChangeListener)) ，而不是[`OnAttachStateChangeListener`](https://developer.android.google.cn/reference/android/view/View.OnAttachStateChangeListener.html?hl=zh-cn) 的一个 setter 方法。`android.databinding.adapters.ListenerUtil` 类帮助记录上一个监听器，这样它们就能被绑定表达式移除。

通过和 `@TargetApi(VERSION_CODES.HONEYCOMB_MR1)` 一起添加 `OnViewDetachedFromWindow` 和 `OnViewAttachedToWindow` 注解，数据绑定的代码生成器就能明白监听器只应当在 Android 3.1（API 版本 12）及更高版本上生成，即 [`addOnAttachStateChangeListener()`](https://developer.android.google.cn/reference/android/view/View.html?hl=zh-cn#addOnAttachStateChangeListener(android.view.View.OnAttachStateChangeListener)) 方法所支持的最低版本。

## 对象的转换

### 自动对象转换

当绑定表达式返回一个 [`Object`](https://developer.android.google.cn/reference/java/lang/Object.html?hl=zh-cn) 时，数据绑定库会选择为属性赋值的方法，该 `Object` 就会被显式转换成被选中的方法的参数类型。这种行为对于使用 [`ObservableMap`](https://developer.android.google.cn/reference/android/databinding/ObservableMap.html?hl=zh-cn) 来存储数据的应用而言非常便利，如下例所示：

```xml
<TextView
   android:text='@{userMap["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content" />
```

> **注意**：您也可以使用一个 `object.key`* 来在映射中访问一个值。例如，上例中的 `@{userMap["lastName"]}` 可以被 `@{userMap.lastName}` 取代。

表达式中的 `userMap` 对象返回的值会被自动显式转换成 `setText(CharSequence)` 方法中找到的参数类型，用来设置 `android:text` 属性的值。如果参数的类型是有歧义的，那么您就必须显示转换该表达式的返回值。

### 自定义的转换

在一些情况下，特定的类型之间必须要实现自定义的转换。例如，视图的 `android:background` 属性期待一个 [`Drawable`](https://developer.android.google.cn/reference/android/graphics/drawable/Drawable.html?hl=zh-cn)，但是 `color` 值期的是一个整型。如下的代码展示了一个期待 `Drawable` 但提供的只有整型的属性：

```xml
<View
   android:background="@{isError ? @color/red : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

当期待 `Drawable` 但返回的只有整型的时候，`int` 应当被转换成 [`ColorDrawble`](https://developer.android.google.cn/reference/android/graphics/drawable/ColorDrawable.html?hl=zh-cn)，这可以求诸一个含有 [`BindingConversion`](https://developer.android.google.cn/reference/android/databinding/BindingConversion.html?hl=zh-cn) 注解的静态方法，如下例所示：

```java
@BindingConversion
public static ColorDrawable convertColorToDrawable(int color) {
    return new ColorDrawable(color);
}
```

然而，绑定表达式中提供的值的类型必须是一致的。您不能在同一个表达式中使用不同的类型，如下例所示：

```xml
<View
   android:background="@{isError ? @drawable/error : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

