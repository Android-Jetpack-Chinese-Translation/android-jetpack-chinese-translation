# 使用 NavigationUI 来更新 UI 组件
> 原文链接：[Update UI components with NavigationUI  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation/navigation-ui)

Navigation 架构组件包含了一个 [`NavigationUI`](https://developer.android.google.cn/reference/androidx/navigation/ui/NavigationUI.html?hl=zh-cn) 的类，该类提供了使用顶部应用栏、导航抽屉和底部导航栏来管理导航的静态方法。

## 监听导航事件

[`NavController`](https://developer.android.google.cn/reference/androidx/navigation/NavController?hl=zh-cn) 是在目的地之间导航的主要手段，它负责把 [`NavHost`](https://developer.android.google.cn/reference/androidx/navigation/NavHost?hl=zh-cn) 的内容替换成新的目的地。在许多情况下，顶部应用栏和其他持久的导航控件（如底部导航栏）等 UI 元素的生命周期比 `NavHost` 要长，因此需要在您导航到其他目的地时被更新。

`NavController` 提供了一个 `OnDestinationChangedListener` 接口，用于在 `NavController` 的[当前目的地](https://developer.android.google.cn/reference/androidx/navigation/NavController?hl=zh-cn#getCurrentDestination%28%29)或参数改变时被调用。您可以使用 [`addOnDestinationChangedListener()`](https://developer.android.google.cn/reference/androidx/navigation/NavController?hl=zh-cn#addOnDestinationChangedListener%28androidx.navigation.NavController.OnDestinationChangedListener%29) 来注册一个新的监听器，如下例所示。请留意：当 `addOnDestinationChangedListener()` 被调用时，如果当前目的地已存在，那它就会被立即发送到您的监听器。

```java
navController.addOnNavigatedListener(new NavController.OnNavigatedListener() {
  @Override
  public void onNavigated(@NonNull NavController navController,
          @NonNull NavDestination destination,
          @Nullable Bundle arguments) {
    textView.setText(destination.getLabel());
  }
});
```

`NavigationUI` 使用 `OnDestinationChangedListener` 来让这些常见的 UI 组件能感知到导航事件。但请注意，您也可以直接只利用 `OnDestinationChangedListener` 来让自定义的 UI 或业务逻辑感知到导航事件。

## 顶部应用栏

[顶部应用栏](https://material.io/design/components/app-bars-top.html) 持久占据您应用的顶部位置，用于显示当前页面的信息和操作。

![顶部应用栏](https://developer.android.google.cn/topic/libraries/architecture/images/top-app-bar.png?hl=zh-cn)

`NavigationUI` 提供了在用户导航时自动更新您顶部应用栏的内容的方法。例如，`NavigationUI` 使用您导航图中目的地的标签来保持顶部应用栏的及时更新。

当您配合着顶部导航栏的（下面将会讨论的）这些方法一起使用 `NavigationUI` 时，目的地就会从（以 `{argName}` 的格式提供给目的地的参数中 的标签就能从被自动填充。

`NavigationUI` 提供了对下列顶部导航栏类型的支持：

* [`工具栏（Toolbar）`](https://developer.android.google.cn/reference/android/widget/Toolbar?hl=zh-cn)
* [`CollapsingToolbarLayout`](https://developer.android.google.cn/reference/android/support/design/widget/CollapsingToolbarLayout?hl=zh-cn)
* [`动作栏（ActionBar）`](https://developer.android.google.cn/reference/android/support/v7/app/ActionBar?hl=zh-cn)

### AppBarConfiguration

`NavigationUI` 使用一个 [`AppBarConfiguration`](https://developer.android.google.cn/reference/androidx/navigation/ui/AppBarConfiguration?hl=zh-cn) 对象来管理您应用左上角的导航按钮的行为。默认情况下，当用户在顶层的目的地时，导航按钮是隐藏的；而当用户在其他的目的地时，导航按钮显示为一个”向上“按钮。

欲将您导航图的起始目的地用作唯一的顶层目的地，您可以创建一个 `AppBarConfiguration` 对象并传入相应的导航图，如下所示：

```kotlin
val appBarConfiguration = AppBarConfiguration(navController.graph)
```

欲将多个目的地定义为顶层目的地，您可以向 `AppBarConfiguration` 的构造器传入一组目的地的 ID，如下所示：

```kotlin
val appBarConfiguration = AppBarConfiguration(setOf(R.id.main, R.id.android))
```

### 创建一个工具栏

欲使用 `NavigationUI` 创建一个工具栏（Toolbar），请首先定义您主 activity 的工具栏，如下所示：

```xml
<LinearLayout>
    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar" />
    <fragment
        android:id="@+id/nav_host_fragment"
        ... />
    ...
</LinearLayout>
```

接着，从您主 activity 的 `onCreate()` 方法中调用 [`setupWithNavController()`](https://developer.android.google.cn/reference/androidx/navigation/ui/NavigationUI.html?hl=zh-cn#setupWithNavController(android.support.v7.widget.Toolbar,%20androidx.navigation.NavController,%20androidx.navigation.ui.AppBarConfiguration))，如下所示：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    setContentView(R.layout.activity_main)

    ...

    val navController = findNavController(R.id.nav_host_fragment)
    val appBarConfiguration = AppBarConfiguration(navController.graph)
    findViewById<Toolbar>(R.id.toolbar)
        .setupWithNavController(navController, appBarConfiguration)
}
```

> **注意**：使用 **Toolbar** 时，Navigation 能自动处理导航按钮的点击事件，因此您不需要覆写 `onSupportNavigateUp()` 方法。

### 包含 CollapsingToolbarLayout

欲在您的工具栏中包含一个 `CollapsingToolbarLayout`，请首先在您的主 activity 中定义工具栏及其外围布局，如下所示：

```xml
<LinearLayout>
    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="@dimen/tall_toolbar_height">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleGravity="top"
            app:layout_scrollFlags="scroll|exitUntilCollapsed|snap">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"/>
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <fragment
        android:id="@+id/nav_host_fragment"
        ... />
    ...
</LinearLayout>
```

接着，从您主 activity 的 `onCreate()` 方法中调用 [`setupWithNavController`](https://developer.android.google.cn/reference/androidx/navigation/ui/NavigationUI.html?hl=zh-cn#setupWithNavController(android.support.v7.widget.Toolbar,%20androidx.navigation.NavController,%20androidx.navigation.ui.AppBarConfiguration))，如下所示：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    setContentView(R.layout.activity_main)

    ...

    val layout = findViewById<CollapsingToolbarLayout>(R.id.collapsing_toolbar_layout)
    val toolbar = findViewById<Toolbar>(R.id.toolbar)
    val navController = findNavController(R.id.nav_host_fragment)
    val appBarConfiguration = AppBarConfiguration(navController.graph)
    layout.setupWithNavController(toolbar, navController, appBarConfiguration)
}
```

### 动作栏

欲添加对默认动作栏的导航支持，请在您的主 activity 的 `onCreate()` 方法中调用 [`setupActionBarWithNavController()`](https://developer.android.google.cn/reference/androidx/navigation/ui/NavigationUI.html?hl=zh-cn#setupActionBarWithNavController(android.support.v7.app.AppCompatActivity,%20androidx.navigation.NavController,%20androidx.navigation.ui.AppBarConfiguration))，如下所示：

```kotlin
private lateinit var appBarConfiguration: AppBarConfiguration
// 请注意，您需要在 `onCreate()` 之外定义 `AppBarConfiguration`，
// 因为您还需要使用它来覆写 `onSupportNavigateUp()`。

...

override fun onCreate(savedInstanceState: Bundle?) {
    ...

    val navController = findNavController(R.id.nav_host_fragment)
    appBarConfiguration = AppBarConfiguration(navController.graph)
    setupActionBarWithNavController(navController, appBarConfiguration)
}
```

接着，覆写 `onSupportNavigateUp()` 来处理”向上“导航。

```kotlin
override fun onSupportNavigateUp(): Boolean {
    return navController.navigateUp(appBarConfiguration) || super.onSupportNavigateUp()
}
```

## 把目的地绑定到菜单项

`NavigationUI` 还提供了用于将目的地绑定到菜单驱动的 UI 组件上的助手方法：[`onNavDestinatinoSelected()`](https://developer.android.google.cn/reference/androidx/navigation/ui/NavigationUI.html?hl=zh-cn#onnavdestinationselected)。该方法接受一个 [`MenuItem`](https://developer.android.google.cn/reference/android/view/MenuItem?hl=zh-cn) 参数和一个持有其绑定的目的地的 [`NavController`](https://developer.android.google.cn/reference/androidx/navigation/NavController?hl=zh-cn) 参数。如果 `MenuItem` 的 `id` 和目的地的 `id` 是相同的，那么 `NavController` 接着就能导航到该目的地。

下面的 XML 定义了一个菜单项和一个 `id` 为 `details_page_fragment` 的目的地：

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    ... >

    ...

    <fragment android:id="@+id/details_page_fragment"
         android:label="@string/details"
         android:name="com.example.android.myapp.DetailsFragment" />
</navigation>
```

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    ...

    <item
        android:id="@id/details_page_fragment"
        android:icon="@drawable/ic_details"
        android:title="@string/details" />
</menu>
```

譬如，若您的菜单是通过 Activity 的 `onCreateOptionsMenu()` 创建的，那么您可以通过覆写 Activity 的 `onOptionItemSelected()` 来调用 `onNavDestinationSelected()`，从而将其菜单项绑定到各个目的地，如下所示：

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    val navController = findNavController(R.id.nav_host)
    return item.onNavDestinationSelected(navController) || super.onOptionsItemSelected(item)
}
```

现在，当一个用户点击 `details_page_fragment` 菜单项时，应用会自动导航到有着相同 `id` 的相应目的地。

## 添加导航抽屉

导航抽屉是一个 UI 面板，用于显示您应用的主导航菜单。当用户点击抽屉图标![](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-drawer-icon.png?hl=zh-cn)、或者从屏幕左边单指滑入时，抽屉就会出现。

![](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-drawer.png?hl=zh-cn)

抽屉图标会在所有使用 `DrawerLayout` 的 *顶层目的地* 上出现。顶层目的地是您应用的根层目的地。这些顶层目的地并不会在应用栏上显示”向上“按钮。

欲添加一个导航抽屉，请首先将一个 [`DrawerLayout`](https://developer.android.google.cn/topic/libraries/architecture/navigation/reference/android/support/v4/widget/DrawerLayout?hl=zh-cn) 定义为根视图。在 `DrawerLayout` 中，添加一个主要内容的 UI 布局和一个包含了导航抽屉的内容的视图。

例如，下面的布局使用了一个含有两个子视图的 `DrawerLayout`：一个 [`NavHostFragment`](https://developer.android.google.cn/reference/androidx/navigation/fragment/NavHostFragment?hl=zh-cn) 用来容纳主要内容、一个 [`NavigationView`](https://developer.android.google.cn/topic/libraries/architecture/navigation/reference/android/support/design/widget/NavigationView.html?hl=zh-cn) 用来容纳导航抽屉：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 使用 DrawerLayout 来作为 activity 的根容器 -->
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <!-- 用来容纳页面的主要内容的布局（导航抽屉会在它上面滑入）-->
    <fragment
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:id="@+id/nav_host_fragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph" />

    <!-- 用来容纳导航抽屉的内容的容器（使用 NavigationView 来简化配置） -->
    <android.support.design.widget.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true" />

</android.support.v4.widget.DrawerLayout>
```

接着，把 [`DrawerLayout`](https://developer.android.google.cn/topic/libraries/architecture/navigation/reference/android/support/v4/widget/DrawerLayout?hl=zh-cn) 传入 `AppBarConfiguration`，从而将其连接到您的导航图，如下所示：

```kotlin
val appBarConfiguration = AppBarConfiguration(navController.graph, drawerLayout)
```

> **注意**：当您使用 `NavigationUI` 时，顶部导航栏助手会根据当前目的地的变动，自动将导航按钮的图标转变成”汉堡按钮“或”向上“。您无须使用 [`ActionBarDrawerToggle`](https://developer.android.google.cn/topic/libraries/architecture/navigation/reference/android/support/v4/app/ActionBarDrawerToggle?hl=zh-cn)。

接着，在您的主 activity 类中，从 `onCreate()` 方法中调用 [`setupWithNavController()`](https://developer.android.google.cn/topic/libraries/architecture/navigation/reference/androidx/navigation/ui/NavigationUI?hl=zh-cn#setupWithNavController(android.support.design.widget.NavigationView,%20androidx.navigation.NavController))，如下所示：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    setContentView(R.layout.activity_main)

    ...

    val navController = findNavController(R.id.nav_host_fragment)
    findViewById<NavigationView>(R.id.nav_view)
        .setupWithNavController(navController)
}
```

## 底部导航栏

`NavigationUI` 还能处理底部导航栏的情况。当用户选择一个菜单项时，`NavController` 会调用 [`onNavDestinationSelected()`](https://developer.android.google.cn/reference/androidx/navigation/ui/NavigationUI?hl=zh-cn#onNavDestinationSelected(android.view.MenuItem,%20androidx.navigation.NavController)) 并自动更新底部导航栏的所选项。

![](https://developer.android.google.cn/topic/libraries/architecture/images/bottom-navigation.png?hl=zh-cn)

欲在您的应用中创建一个底部导航栏，请首先在主 activity 中定义它，如下所示：

```xml
<LinearLayout>
    ...
    <fragment
        android:id="@+id/nav_host_fragment"
        ... />
    <android.support.design.widget.BottomNavigationView
        android:id="@+id/bottom_nav"
        app:menu="@menu/menu_bottom_nav" />
</LinearLayout>
```

接着，在您的主 activity 类中，从 `onCreate()` 方法中调用 [`setupWithNavController()`](https://developer.android.google.cn/topic/libraries/architecture/navigation/reference/androidx/navigation/ui/NavigationUI?hl=zh-cn#setupWithNavController(android.support.design.widget.NavigationView,%20androidx.navigation.NavController))，如下所示：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    setContentView(R.layout.activity_main)

    ...

    val navController = findNavController(R.id.nav_host_fragment)
    findViewById<BottomNavigationView>(R.id.bottom_nav)
        .setupWithNavController(navController)
}
```
