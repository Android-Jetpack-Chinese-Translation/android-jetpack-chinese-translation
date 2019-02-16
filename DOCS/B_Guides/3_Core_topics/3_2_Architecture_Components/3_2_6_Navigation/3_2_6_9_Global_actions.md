# 全局动作
> 原文链接：[Global actions  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation/navigation-global-action)

您可以使用 **全局动作（global action）** 来创建一个供多个目的地共用的通用动作。例如，您可能想让不同目的地中的按钮都能导航到相同的主界面。

在导航编辑器中，全局动作是用一个指向该目的地的小箭头表示的，如图一所示：

![图一：指向某个嵌套导航图的全局动作](https://developer.android.google.cn/topic/libraries/architecture/images/navigation-global-action.png?hl=zh-cn)
**图一：**指向某个嵌套导航图的全局动作

## 创建全局动作

欲创建一个全局动作，请按以下步骤操作：

1. 在 **Graph Editor** 中，单击一个目的地以使其高亮。
2. 右键单击该目的地来打开 context 菜单。
3. 选择 **Add Action > Global**，一个箭头![](https://developer.android.google.cn/studio/images/buttons/navigation-globalaction.png?hl=zh-cn)就会在该目的地的左边出现。
4. 单击 **Text** 标签页来切换到 XML 文本视图。该全局动作的 XML 代码如下所示：

	```xml
	<?xml version="1.0" encoding="utf-8"?>
	<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
	            xmlns:tools="http://schemas.android.com/tools"
	            xmlns:android="http://schemas.android.com/apk/res/android"
	            android:id="@+id/main_nav"
	            app:startDestination="@id/mainFragment">
	
	  ...
	
	  <action android:id="@+id/action_global_mainFragment"
	          app:destination="@id/mainFragment"/>
	
	</navigation>
	```

## 使用全局动作

欲在您的代码中使用全局动作，请将该全局动作的资源 ID 传入每个 UI 元素的 [`navigate()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#navigate(int)) 方法，如下所示：

```java
viewTransactionsButton.setOnClickListener(new View.OnClickListener() {
   @Override
   public void onClick(View view) {
       Navigation.findNavController(view).navigate(R.id.action_global_mainFragment);
   }
});
```

## 和 Safe Args 一起使用全局动作

欲了解更多和 Safe Args 一起使用全局动作的信息，请参阅[在目的地之间传递数据](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_6_Navigation/3_2_6_5_Pass_data_between_destinations.md)。


