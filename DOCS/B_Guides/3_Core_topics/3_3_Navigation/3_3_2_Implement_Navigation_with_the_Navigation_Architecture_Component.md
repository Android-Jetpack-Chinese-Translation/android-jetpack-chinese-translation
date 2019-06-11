# 用 Navigation 架构组件来实现导航
> 原文链接：[Implement navigation with the Navigation Architecture Component  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation/navigation-implementing)

Android Jetpack 架构组件提供了一系列 Navigation 组件来为您处理琐碎细节，从而让应用内导航易于实现。

使用 Navigation，您可以创建 *导航图*：一个用来表示您应用内各自 *目的地* 节点及其之间 *动作* 的 XML 资源文件。

图一展示了一个样例应用的导航图的可视化表示。该应用含有六个页面，彼此之间用五个动作相连接。

![图一：一个导航图](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-graph_2x.png?hl=zh-cn)
**图一：** 一个导航图

*目的地* 是您应用中任何一个用户能导航到的地方。尽管目的地常常是代表特定页面的 `Fragment`，Navigation 组件也支持下列其他目的地类型：

* Activity。
* 导航图（graph）和子图（subgraph）：当目的地是图/子图时，用户被导航到该图/子图的起始目的地。
* [自定义的目的地类型](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_3_Navigation/3_3_6_Add_support_for_new_destination.md)

> **注意**：Navigation 组件是为有一个主 Activity 和多个 Fragment 目的地的应用所涉及的。主 Activity 持有导航图，并负责在需要的时候切换目的地。在一个有着多个 Activity 目的地的应用中，每个额外的 Activity 持有其各自的导航图。欲了解更多信息，请参阅下面的“修改 Activity 并使其持有导航”。

## 为您的项目建立导航

> **注意**：欲在 Android Studio 中使用 Navigation 架构组件，您必须使用 Android Studio 3.3 或更高版本。

欲为您的项目添加 Navigation 支持，请将下面的代码加入您应用的 `build.gradle` 文件：

```gradle
dependencies {
    def nav_version = "1.0.0-beta01"

    implementation "android.arch.navigation:navigation-fragment:$nav_version" // Kotlin 请使用 -ktx 版本
    implementation "android.arch.navigation:navigation-ui:$nav_version" // Kotlin 请使用 -ktx 版本
}
```

欲了解更多有关为您的项目添加其他架构组件的信息，请参阅[为您的项目添加 Android 架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_2_Adding_Components_to_your_Project.md)。

## 创建导航图

欲为您的项目添加一个导航图，请按如下操作：

1. 在 Project 窗口中，右键单击 `res` 目录并选择 **New > Android Resource File**，您将会看到 **New Resource File** 对话框出现。
2. 在 **File Name** 栏键入一个名字，例如：`nav_graph`。
3. 从 **Resource type** 下拉列表中选择 **Navigation**。
4. 单击 **OK**，您将会看到如下内容：
	1. 一个 `navigation` 资源目录被创建到 `res` 目录。
	2. 一个 `nav_graph.xml` 文件被创建到上述 `navigation` 目录。
	3. 该 `nav_graph.xml` 文件在导航编辑器中被打开，它含有您的导航图。
5. 单击 **Text** 标签页来切换到 XML 文本视图。您应看到一个空白的导航图，如下例所示：

	```xml
	<?xml version="1.0" encoding="utf-8"?>
	<navigation xmlns:android="http://schemas.android.com/apk/res/android">
	</navigation>
	```
6. 单击 **Design** 来返回到导航编辑器中。

### 参观导航编辑器

在导航编辑器中，您可以可视化地编辑导航图，而无须直接编辑其底层的 XML。

![图二：导航编辑器](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-editor_2x.png?hl=zh-cn)
**图二：** 导航编辑器

1. **目的地**：列出您的导航宿主以及**图编辑器**中的所有目的地。
2. **图编辑器**：含有您导航图的一个可视化表示。
3. **属性**：展示了导航图中当前所选项目的属性。

### 创建目的地

创建导航图的第一步，是为您的应用识别并创建各自的目的地。您可以为一个现有项目中的 Activity 和 Fragment 创建目的地，也可以先创建一个目的地占位符，之后再用一个 Activity 或 Fragment 来替代它。

如果您已经有了一个想要添加到导航图中的 Activity 或 Fragment，或者只是想要添加一个占位符，请单击 **New Destination**![](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-new-destination-icon.png?hl=zh-cn)，然后单击下拉菜单中的相应 Activity、Fragment 或占位符。您现在应该能看到在 **Design** 视图中的该目的地的预览、以及在您导航图的 **Text** 视图中的相应 XML 代码。

欲创建一个新的目的地类型，请按如下操作：

1. 在导航编辑器中，单击 **New Destination**![](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-new-destination-icon.png?hl=zh-cn)，并单击 **Create blank destination**。
2. 在新出现的 **New Android Component** 对话框中，输入一个 **Fragment Name** 用作 Fragment 的类名。
3. 欲使 Android Studio 为该 Fragment 创建一个相应的布局文件，请勾选 **Create layout XML** 旁边的多选框，并在 **Fragment Layout Name** 栏中键入该资源的文件名。
4. 在 **Source Language** 下拉菜单中，选择 Java 或 Kotlin 用作该类源文件的语言。
5. 单击 **Finish**。

新的目的地类型会在导航编辑器中出现。Android Studio 还会用上面指定的语言创建一个相应的类文件；如果指明创建布局的话，还会为其创建一个相应的布局元文具。

![图三：一个目的地和一个占位符](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-destination-and-placeholder_2x.png?hl=zh-cn)
**图三：** 一个目的地和一个占位符

您可以单击任何一个目的地来选中它。当您选中一个目的地的时候，如下的属性出现在 **Attributes** 面板中：

* **Type** 字段：该目的地是否已在您的源代码中以一个 Fragment 或 Activity 实现。
* **Label** 字段：该目的地 XML 布局文件的文件名。
* **ID** 字段：该目的地的 ID，用于在代码中引用该目的地。
* **Class** 下拉菜单：该目的地所对应的类名。您可以单击该下拉菜单来把该类绑定到另一个目的地上。

> **注意**：占位符并不和任何类绑定。请务必在运行您的应用之前更改其绑定的类。

单击 **Text** 标签页来显示您导航图的 XML 视图。该 XML 含有该目的地的 `id`、`name`、`label` 和 `layout` 属性，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    app:startDestination="@id/blankFragment">
    <fragment
        android:id="@+id/blankFragment"
        android:name="com.example.cashdog.cashdog.BlankFragment"
        android:label="Blank"
        tools:layout="@layout/fragment_blank" />
</navigation>
```

### 连接目的地

欲连接不同的目的地，您必须含有至少两个目的地。如下的 XML 展示了一个含有两个空白目的地的导航图：

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    app:startDestination="@id/blankFragment">
    <fragment
        android:id="@+id/blankFragment"
        android:name="com.example.cashdog.cashdog.BlankFragment"
        android:label="fragment_blank"
        tools:layout="@layout/fragment_blank" />
    <fragment
        android:id="@+id/blankFragment2"
        android:name="com.example.cashdog.cashdog.BlankFragment2"
        android:label="Blank2"
        tools:layout="@layout/fragment_blank_fragment2" />
</navigation>
```

目的地是用动作来连接的。欲连接两个目的地，请按如下操作：

1. 在 **Design** 标签页中，将指针悬浮在您想要用户开始导航的目的地的右侧，一个圆圈会出现。

	![图四：动作连接圆圈](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-actioncircle_2x.png?hl=zh-cn)
	**图四：** 动作连接圆圈
	
2. 单击该圆圈，将您的指针拖动到您想让用户导航到的目的地，然后释放指针。一条带有箭头的线将会被画出，用于表示两个目的地之间的导航。

	![图五：连接的目的地](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-connected_2x.png?hl=zh-cn)	
	**图五：** 连接的目的地
	
3. 单击该箭头来高亮其动作。如下的属性将出现在 **Attributes** 面板中：
	* **Type** 字段：”Action“。
	* **ID** 字段：该动作的 ID。
	* **Destination** 字段：目的地 Activity 或 Fragment 的 ID。

4. 单击 **Text** 标签页来切换到 XML 视图。一个动作元素会被添加到源目的地。该动作含有一个 ID 属性和一个含有下一个目的地的 ID 的目的地属性，如下所示：

	```xml
	<?xml version="1.0" encoding="utf-8"?>
	<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
	    xmlns:tools="http://schemas.android.com/tools"
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    app:startDestination="@id/blankFragment">
	    <fragment
	        android:id="@+id/blankFragment"
	        android:name="com.example.cashdog.cashdog.BlankFragment"
	        android:label="fragment_blank"
	        tools:layout="@layout/fragment_blank" >
	        <action
	            android:id="@+id/action_blankFragment_to_blankFragment2"
	            app:destination="@id/blankFragment2" />
	    </fragment>
	    <fragment
	        android:id="@+id/blankFragment2"
	        android:name="com.example.cashdog.cashdog.BlankFragment2"
	        android:label="fragment_blank_fragment2"
	        tools:layout="@layout/fragment_blank_fragment2" />
	</navigation>	
	```
	
### 指定一个页面来作为起始目的地

导航编辑器使用一个房子形状的图标![](https://developer.android.google.cn/studio/images/buttons/navigation-house.png?hl=zh-cn) 来表示用户打开您的应用后看到的第一个页面，即：**起始目的地**。

欲选择一个不同的起始目的地，请按如下操作：

1. 在 **Design** 标签页中，单击新的起始目的地，并让其高亮显示。
2. 单击 **Assign start destination** 按钮![](https://developer.android.google.cn/studio/images/buttons/navigation-house.png?hl=zh-cn)。或者，您也可以右键单击一个目的地，并单击 **Set as Start Destination**。

## 修改 Activity 并使其持有导航

一个 Activity 在 [`NavHost`](https://developer.android.google.cn/reference/androidx/navigation/NavHost.html?hl=zh-cn) 中持有其应用的导航。`NavHost` 是一个空的容器，当用户您的应用的不同页面间跳转时，目的地会在该容器中交换。

Navigation 组件的默认 [`NavHost`](https://developer.android.google.cn/reference/androidx/navigation/NavHost.html?hl=zh-cn) 实现是 [`NavHostFragment`](https://developer.android.google.cn/reference/androidx/navigation/fragment/NavHostFragment.html?hl=zh-cn)。

### 使用布局编辑器来添加一个 NavHostFragment

您可以使用[布局编辑器](https://developer.android.google.cn/studio/write/layout-editor?hl=zh-cn)，按如下操作来为一个 Activity 添加一个 `NavHostFragment`：

1. 创建一个导航图，如果您还没做的话。
2. 在您的项目文件列表中，双击您 Activity 的 XML 布局文件来在布局编辑器中打开它。
3. 在 **Palette** 面板中，选择 **Containers** 类别。或者，您也可以搜索 `NavHostFragment`。
4. 把 `NavHostFragment` 视图拖动到您的 Activity 上。
5. 在新出现的 **Navigation Graphs** 对话框中，选择相应的导航图来绑定到该 `NavHostFragment` 上，然后单击 **OK**。

回到 **Text** 视图下，注意 Android Studio 已经添加了如下所示的 XML 代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"

        app:defaultNavHost="true"
        app:navGraph="@navigation/mobile_navigation" />

</android.support.constraint.ConstraintLayout>
```

`app:defaultNavHost="true"` 属性确保您的 `NavHostFragment` 能截获系统的返回键事件。您还可以通过覆写 [`AppCompatActivity.onSupportNavigateUp()`](https://developer.android.google.cn/reference/android/support/v7/app/AppCompatActivity?hl=zh-cn) 方法，并调用 [`NavController.navigateUp()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#navigateUp()) 来实现这一功能，如下所示：

```java
@Override
public boolean onSupportNavigateUp() {
    return Navigation.findNavController(this, R.id.nav_host_fragment).navigateUp();
}
```

### 用代码创建 NavHostFragment

您还可以使用 [`NavHostFragment.create()`](https://developer.android.google.cn/reference/androidx/navigation/fragment/NavHostFragment?hl=zh-cn#create) 来为某个给定的导航图资源创建 `NavHostFragment`，如下例所示：

```java
NavHostFragment finalHost = NavHostFragment.create(R.navigation.example_graph);
getSupportFragmentManager().beginTransaction()
    .replace(R.id.nav_host, finalHost)
    .setPrimaryNavigationFragment(finalHost) // 这等价于 app:defaultNavHost="true"
    .commit();
```

## 将目的地绑定到 UI 控件上

对目的地的导航是通过 [`NavController`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn) 类实现的。您可以使用下列静态方法来获取一个 [`NavController`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn) 实例：

* [`NavHostFragment.findNavController(Fragment)`](https://developer.android.google.cn/reference/androidx/navigation/fragment/NavHostFragment.html?hl=zh-cn#findNavController(android.support.v4.app.Fragment))
* [`Navigation.findNavController(Activity, @IdRes int viewId)`](https://developer.android.google.cn/reference/androidx/navigation/Navigation.html?hl=zh-cn#findNavController(android.app.Activity,%20int))
* [`Navigation.findNavController(View)`](https://developer.android.google.cn/reference/androidx/navigation/Navigation.html?hl=zh-cn#findNavController(android.view.View))

当你获取一个 `NavController` 之后，使用其 [`navigate()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#navigate(int)) 方法来导航到某个目的地。[`navigate()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#navigate(int)) 方法接受一个资源 ID 作为参数，该 ID 可以是该目的地在导航图中的 ID，也可以是一个动作的 ID。使用动作 ID 相较于使用目的地 ID 而言有一些好处，譬如为您的导航绑定转场动画。欲了解更多有关转场动画的信息，请参阅下面的”在目的地之间创建转场动画“。

如下代码展示了如何导航到 `ViewTransactionsFragment`：

```java
viewTransactionsButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Navigation.findNavController(view).navigate(R.id.viewTransactionsAction);
    }
});
```

Android 操作系统维护着一个 [`回退栈`](https://developer.android.google.cn/guide/components/activities/tasks-and-back-stack?hl=zh-cn)，用于储存最新访问的目的地。您应用的起始目的地会在用户启动应用时被放入栈中。每一次调用 [`navigate()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#navigate(int)) 都会把另一个目的地放到栈顶；反过来说，点击”向上“和”向后“按钮会分别调用 [`NavController.navigateUp()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#navigateUp()) 和 [`NavController.popBackStack()`](NavController.popBackStack())，从而将栈顶的目的地 pop 出栈。

对按钮而言，您也可以使用 `Navigation` 类的 [`createNavigationOnClickListener()`](https://developer.android.google.cn/reference/androidx/navigation/Navigation.html?hl=zh-cn#createNavigateOnClickListener(int)) 便利方法来导航到某个目的地：

```java
button.setOnClickListener(Navigation.createNavigateOnClickListener(R.id.next_fragment, null));
```

欲处理其他常见 UI 组件的情况，例如顶部应用栏和底部导航栏，请参阅 [使用 NavigationUI 来更新 UI 组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_3_Navigation/3_3_3_Update_UI_components_with_NavigationUI.md)。

## 在目的地之间创建转场动画

Navigation 组件能让您在目的地之间轻松添加淡入、淡出等转场动画。欲添加转场动画，请按如下操作：

1. 创建您的动画资源文件。Navigation 支持属性（property）动画和视图（view）动画。欲了解更多信息，请参阅 [动画资源](https://developer.android.google.cn/guide/topics/resources/animation-resource?hl=zh-cn)。
2. 在导航编辑器中的 **Design** 标签页，单击您想让转场发生的动作。
3. 在 **Attributes** 面板的 **Transitions** 部分，单击 **Enter** 旁边的向下箭头来显示您项目中所有可用的转场动画。
4. 选择一个您想在用户到达目的地时发生的转场动画。
5. 回到 **Transitions** 部分，单击 **Exit** 旁边的向下箭头，然后选择一个您想在用户离开目的地时发生的转场。
6. 单击 **Text** 标签页来切换到 XML 文本视图。转场的 XML 代码会出现在指定动作的 `<action>` 元素中。该动作被嵌入到转场发生前的活跃目的地的 XML 中。

	在下面的例子中，`specifyAmountFragment` 是活跃的目的地，因此它含有带转场动画的动作：
	
	```xml
	<fragment
	    android:id="@+id/specifyAmountFragment"
	    android:name="com.example.buybuddy.buybuddy.SpecifyAmountFragment"
	    android:label="fragment_specify_amount"
	    tools:layout="@layout/fragment_specify_amount">
	    <action
	        android:id="@+id/confirmationAction"
	        app:destination="@id/confirmationFragment"
	        app:enterAnim="@anim/slide_in_right"
	        app:exitAnim="@anim/slide_out_left"
	        app:popEnterAnim="@anim/slide_in_left"
	        app:popExitAnim="@anim/slide_out_right" />
	 </fragment>
	```
	
	在这个例子中，用户到达目的地（`enterAnim`、`exitAnim`）和离开该目的地（`popEnterAnim`、`popExitAnim`）时都有转场动画。
	
### 在目的地之间添加共享元素转场动画

除了转场动画之外，Navigation 还支持在目的地之间添加共享元素转场动画（Shared element transitions）。

普通的动画是通过导航的 XML 文件实现的，而共享元素转场动画则不同：由于要引用将要被共享的 `View` 实例，它必须在代码中实现。

每种目的地都通过 [`Navigator.Extras`](https://developer.android.google.cn/reference/androidx/navigation/Navigator.Extras?hl=zh-cn) 接口的一个子类来实现该 API。`Extras` 将会被传入 [`navigate()`](https://developer.android.google.cn/reference/androidx/navigation/NavController?hl=zh-cn#navigate(int,%20android.os.Bundle,%20androidx.navigation.NavOptions,%20androidx.navigation.Navigator.Extras)) 的调用。

#### Fragment 目的地的共享元素转场动画

[`FragmentNavigator.Extras`](https://developer.android.google.cn/reference/androidx/navigation/fragment/FragmentNavigator.Extras?hl=zh-cn) 让您能把共享的元素传入针对 Fragment 目的地的 `navigate()` 方法调用中，如下例所示：

```java
FragmentNavigator.Extras extras = new FragmentNavigator.Extras.Builder()
    .addSharedElement(imageView, "header_image")
    .addSharedElement(titleView, "header_title")
    .build();
    
Navigation.findNavController(view).navigate(R.id.details,
    null, // 参数的 Bundle
    null, // NavOptions
    extras);
```

### Activity 目的地的共享元素转场动画

Activity 依赖于 [`ActivityOptionsCompat`](https://developer.android.google.cn/reference/android/support/v4/app/ActivityOptionsCompat?hl=zh-cn) 来控制共享元素转场动画，如下例所示：

```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(this,
        Pair.create(imageView, "header_image"),
        Pair.create(titleView, "header_title"));

ActivityNavigator.Extras extras = new ActivityNavigator.Extras.Builder()
    .setActivityOptions(options)
    .build();
    
Navigation.findNavController(view).navigate(R.id.details,
    null, // 参数的 Bundle
    null, // NavOptions
    extras);
```

欲了解更多信息，请参阅[使用共享元素转场动画来启动一个 activity](https://developer.android.google.cn/training/transitions/start-activity?hl=zh-cn#start-transition)的文档。


