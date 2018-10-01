# 数据绑定概览
> 原文链接：[Data Binding Library  |  Android Developers](https://developer.android.google.cn/topic/libraries/data-binding/)

数据绑定库是一个让您能够声明式（而非命令式）地把布局文件中的 UI 组件绑定到您应用中的数据源的支持库。

布局文件常常是在 activity 中被定义的，并附有调用 UI 架构方法的代码。例如，如下的代码示例调用 [`findViewById()`](https://developer.android.google.cn/reference/android/app/Activity.html#findViewById(int)) 来找到特定的 [`TextView`](https://developer.android.google.cn/reference/android/widget/TextView.html) 部件，并将其绑定到 `viewModel` 变量的 `userName` 字段。

```java
TextView textView = findViewById(R.id.sample_text);
textView.setText(viewModel.getUserName());
```

下面的示例展示了如何使用数据绑定库来把文本直接赋值给布局文件中的部件。这能避免调用上述例子中的任何 Java 代码。请注意赋值表达式中的 `@{}` 句法结构：

```xml
<TextView
    android:text="@{viewmodel.userName}" />
```

绑定布局文件中的组件能帮助您从 activity 中移除许多 UI 框架的调用，从而使其更简洁且易于维护。这也能提升您应用的性能，避免内存泄漏和空指针异常。

请参阅下面的内容来学习如何在您的应用中使用数据绑定库。欲了解更多示范代码，请参阅 [Android 数据绑定库示例](https://github.com/googlesamples/android-databinding)。

## 资源

#### [开始动手](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_2_Get_Started.md)

学习如何准备您用来使用数据绑定的开发环境，包括 Android Studio 中的对数据绑定代码的支持。

#### [布局文件和绑定表达式](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_3_Layouts_and_binding_expressions.md)

数据绑定的表达式语言允许您直接将变量绑定到布局文件中的视图。数据绑定库能够自动生成完成绑定所需的类，而且还提供了导入、变量和包含等可以在布局文件中使用的特性。

数据绑定库的这些特性和您现有的布局文件能够无缝并存。例如，可在表达式中使用的绑定变量是在一个数据元素中定义的，而该数据元素则是 UI 布局根元素的兄弟节点。两个元素都被包含在一个 `layout` 标签中，如下例所示：

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="viewmodel"
            type="com.myapp.data.ViewModel" />
    </data>
    <ConstraintLayout... /> <!-- UI 布局的根元素 -->
</layout>
```

#### [与可观测的数据对象一起使用](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_4_Work_with_observable_data_objects.md)

数据绑定库提供了帮助您轻松观测数据变化的类和接口。您无须操心数据变化时的 UI 更新：数据绑定库允许您让对象、字段或集合实现可观测接口。

#### [自动生成的绑定类](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_5_Generated_binding_classes.md)

数据绑定库能自动生成绑定类，用于获取布局的变量和视图。该小节展示了如何使用并定制这些自动生成的绑定类。

#### [绑定适配器](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_6_Binding_adapters.md)

对于每个布局的绑定表达式而言，都有一个绑定适配器来执行所需的框架方法调用，用于设置相应的属性或监听器。例如，绑定适配器能够处理的情况有：调用 `setText()` 方法来设置文本属性、或者调用 `setOnClickListener()` 方法来为点击事件添加监听器，等等。`android.databinding.adapters` 包中已为您提供了最常用的绑定适配器，例如该小节中用到的 `android:text` 属性的适配器。欲查看常见的绑定适配器的清单，请参阅[适配器](https://android.googlesource.com/platform/frameworks/data-binding/+/master/extensions/baseAdapters/src/main/java/android/databinding/adapters/)。您也可以创建定制的适配器，如下例所示：

```java
@BindingAdapter("app:goneUnless")
public static void goneUnless(View view, Boolean visible) {
    view.visibility = visible ? View.VISIBLE : View.GONE;
}
```

#### [将布局绑定到架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_7_Bind_layout_views_to_Architecture_Components.md)

Android 支持库包含了[架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_1_Overview.md)，用于帮您设计稳健、可测试、易于维护的应用。您可以把数据绑定库同架构组件组合使用，这样可以使您的 UI 开发更轻松。

#### [双向数据绑定](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_3_Data_Binding_Library/3_2_3_8_Two-way_data_binding.md)

数据绑定库支持双向数据绑定。这种类型的绑定所使用的注释符号能够接某个属性的数据变化，并在同时监听用户对该属性所作出的更新。

