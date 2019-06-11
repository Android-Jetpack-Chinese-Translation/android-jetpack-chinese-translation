# 在目的地之间传递数据
> 原文链接：[Pass data between destinations  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation/navigation-pass-data)

通过给目的地定义参数的方式，Navigation 允许您为导航操作附加数据。例如，一个用户个人主页的目的地就可以接受一个用户 ID 作为参数，来决定具体显示哪一个用户。

总的来说，您在目的地之间应尽可能少地只传递必需的数据。例如，您不应把数据对象直接传递过去，而是应该传递一个键来检索该对象，因为 Android 准备给保存的状态的全部空间是有限的。如果您需要传递大量数据，请考虑使用[在 fragment 之间分享数据](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_ViewModel.md)中提及的 [`ViewModel`](https://developer.android.google.cn/reference/androidx/lifecycle/ViewModel?hl=zh-cn)。

## 定义目的地参数

欲在目的地之间传递数据，请按下列步骤，首先为接受参数的目的地定义这些参数：

1. 在[导航编辑器](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_3_Navigation/3_3_2_Implement_Navigation_with_the_Navigation_Architecture_Component.md)中，单击将要接受参数的目的地。
2. 在 **Attributes** 面板中，单击 **Add (+)**。
3. 在新出现的 **Add Argument Link** 窗口中，输入参数的名字、类型、可否为空值，以及（如果需要的话）一个默认值。
4. 单击 **Add**。请注意现在 **Attributes** 面板的 **Arguments** 列表中已出现该参数。
5. 接着，点击将用户引导到该目的地的动作。在 **Attributes** 面板中，您现在应该已经能在 **Argument Default Values** 部分中看到新添加的参数了。
6. 您还可以在 XML 代码中看到该参数。单击 **Text** 标签页来切换 XML 视图，请注意您的参数现在已被添加到接受它的目的地了。如下例所示：
	
	```xml
	 <fragment android:id="@+id/myFragment" >
	     <argument
	         android:name="myArg"
	         app:argType="integer"
	         android:defaultValue="0" />
	 </fragment>
	```

### 在动作中覆写一个目的地参数

在目的地的层级上的参数及其默认值是被所有导航到该目的地的动作所共用的。必要的情况下，您可以通过在动作的层级上定义参数的方式，来覆写一个目的地参数的默认值，或者在缺省的情况下为其赋值。该参数必须和目的地中定义的参数有着相同的名字和类型。

如下的 XML 声明了一个动作，该动作覆写了上面例子中的目的地层级的参数：

```xml
<action android:id="@+id/startMyFragment"
    app:destination="@+id/myFragment">
    <argument
        android:name="myArg"
        app:argType="integer"
        android:defaultValue="1" />
</action>
```

## 使用 Safe Args 来类型安全地传递数据

Navigation 架构组件含有一个名为 **Safe Args** 的 Gradle 插件，它能生成简单的对象和生成器，用于类型安全（type-safe）地访问目的地/动作层级定义的参数。

>**注意**：欲为您的 Android 项目添加 Safe Args 支持，请参阅 [Navigation 发布指南](https://developer.android.google.cn/jetpack/androidx/releases/navigation?hl=zh-cn#safeargs) 中的声明依赖的说明。

在启用 Safe Args 之后，您就得到了针对每个有发送和接收目的地的动作而自动生成的含有了类型安全的方法的代码。这些代码如下所示：

* 针对每个有动作发出的目的地都会生成一个类。该类的类名是其发出的目的地 + ”Directions“。例如，如果发出动作的该目的地是一个名为 `SpecifyAmountFragment` 的 fragment，那么自动生成的类就叫做 `SpecifyAmountFragmentDirections`。该类针对上述目的地的每个动作都定义了一个方法。
* 针对每个用来传递参数的动作都会生成一个内部类，且该类的类名取自其动作。例如，如果一个动作名为 `confirmationAction`，那么自动生成的内部类就叫做 `ConfirmationAction`。
* 针对接收的参数目的地会生成一个类，且该类的类名取自其目的地 + ”Args“。例如，如果该目的地 fragment 名为 `ConfirmationFragment`，那么自动生成的类就叫做 `ConfirmationFragmentArgs`。使用该类的 `fromBundle()` 方法来获取参数。

下例展示了如何使用这些方法来设置一个参数并将其传入 [`navigate()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#navigate(int)) 方法：

```java
@Override
public void onClick(View view) {
   EditText amountTv = (EditText) getView().findViewById(R.id.editTextAmount);
   int amount = Integer.parseInt(amountTv.getText().toString());
   ConfirmationAction action =
           SpecifyAmountFragmentDirections.confirmationAction()
   action.setAmount(amount)
   Navigation.findNavController(view).navigate(action);
}
```

在您接收参数的目的地的代码中，使用 [`getArguments()`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html?hl=zh-cn#getArguments()) 方法来获取 bundle 并使用其包含的数据。当您使用 `-ktx` 依赖关系时，Kotlin 用户还可以使用 `by navArgs()` 的属性委托来访问这些参数。

```java
@Override
public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
    TextView tv = view.findViewById(R.id.textViewAmount);
    int amount = ConfirmationFragmentArgs.fromBundle(getArguments()).getAmount();
    tv.setText(amount + "")
}
```

```kotlin
val args: ConfirmationFragmentArgs by navArgs() // 委托

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    val tv: TextView = view.findViewById(R.id.textViewAmount)
    val amount = args.amount
    tv.text = amount.toString()
}
```

### 和全局动作一起使用 Safe Args

当您和[全局动作（global action）](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_3_Navigation/3_3_9_Global_actions.md)一起使用 Safe Args 时，您必须为根节点 `<navigation>` 提供一个 `android:id` 的值，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
            xmlns:tools="http://schemas.android.com/tools"
            xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@+id/main_nav"
            app:startDestination="@id/mainFragment">

    ...

</navigation>
```

Navigation 组件会基于该 `android:id` 值、为 `<navigation>` 元素自动生成一个 `Directions` 类。例如，如果您的 `<navigation>` 节点有一个 `android:id=@+id/main_nav` 的属性，那么自动生成的类就叫做 `MainNavDirections`。`<navigation>` 中的所有目的地都有自动生成的方法，用于访问所有相连的全局动作，如上小节所述。

## 在目的地之间用 Bundle 对象来传递数据

即使您没有使用 Gradle，您仍能在目的地之间用 `Bundle` 对象来传递数据。创建一个 `Bundle` 对象并使用 [`navigate()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#navigate(int)) 将其传入目的地，如下所示：

```kotlin
var bundle = bundleOf("amount" to amount)
view.findNavController().navigate(R.id.confirmationAction, bundle)
```

在您接收参数的目的地的代码中，使用 [`getArguments()`](https://developer.android.google.cn/reference/android/support/v4/app/Fragment.html?hl=zh-cn#getArguments()) 方法来获取 bundle 并使用其包含的数据。

```kotlin
val tv = view.findViewById<TextView>(R.id.textViewAmount)
tv.text = arguments.getString("amount")
```



