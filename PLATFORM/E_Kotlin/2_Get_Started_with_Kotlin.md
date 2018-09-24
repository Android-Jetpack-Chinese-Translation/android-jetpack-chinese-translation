# 开始动手在 Android 中使用 Kotlin 
> 原文链接：[Get Started with Kotlin on Android  |  Android Developers](https://developer.android.google.cn/kotlin/get-started)

[Android Studio](https://developer.android.google.cn/studio/) 3.0 或更高版本对于 Kotlin 有着完整的支持，因此您可以轻易地创建使用 Kotlin 的[新项目](https://developer.android.google.cn/studio/projects/create-project)、为您现有的项目[添加 Kotlin 文件](https://developer.android.google.cn/studio/projects/add-kotlin)，或者[将 Java 代码转换成 Kotlin](https://developer.android.google.cn/studio/projects/add-kotlin#convert-to-kotlin-code)。这样一来，您你就能使用 Android Studio 为 Kotlin 提供的所有工具，例如代码自动补全（autocomplete）、代码风格检查（lint checking）、重构和调试，等等。

#### [开始动手在 Android 中使用 Kotlin —— YouTube](https://www.youtube.com/watch?v=czKo-jPVweg)

忍不住想上手来一发？请查看我们的 [Kotlin 案例](https://developer.android.google.cn/samples/index.html?language=kotlin)。欲了解有关 Kotlin 编程语言的更多信息，请参阅[学习 Kotlin 的资源](https://developer.android.google.cn/kotlin/resources.html)。

## 为现有的应用添加 Kotlin 代码

欲获得使用 Kotlin 的技能和自信，我们推荐一条循序渐进的方法：

1. **从使用 Kotlin [编写测试](https://developer.android.google.cn/studio/test/#add_a_new_test)开始**：测试对于检查代码在修改后是否工作有着重要的作用，让您能更安心地重构代码。而在把 Java 代码转换成 Kotlin 时，测试就更至关重要了。由于测试在打包阶段并不和您的应用绑定，您可以放心大胆地通过它们向代码库中添加 Kotlin 代码。
2. **使用 Kotlin 编写新的代码**：在将现有 Java 代码转换成 Kotlin 之前，请试着[使用 Kotlin向您的应用中添加少量新代码](https://developer.android.google.cn/studio/projects/add-kotlin)。譬如，您可以从一个不大的类或者顶层的助手函数开始。请务必为这些 Kotlin 代码添加相关的注释，以确保它们可以和 Java 代码协同互用。
3. **将现有代码更新为 Kotlin**：一旦您已经熟悉了使用 Kotlin 编写新代码，就可以把您现有的 Java 代码转换成 Kotlin 了。您可能需要将少量使用 Java 代码实现的功能提取出来、并转化成 Kotlin 类和顶层的函数。

先试试 Android Studio 中能够将 Java 文件转成 Kotlin 的 [代码转换器](https://developer.android.google.cn/studio/projects/add-kotlin#convert-to-kotlin-code)吧。您还可以把剪贴板里的 Java 代码在拷贝到 Kotlin 文件中的时候进行转化。

## Android API 和 Kotlin 范例

Kotlin 提供了[和 Java 的完全的互用性（interoperability）](https://kotlinlang.org/docs/reference/java-interop.html)，因此调用 Android API 常常看起来和 Java 代码完全一致。此外，您现在还能把这种方法调用同 Kotlin 的语义特性结合起来。

我们还在努力为所有的 Android API 文档提供提到的 Kotlin 参考。您可以在 [Android API 文档概述](https://developer.android.google.cn/reference/)中找到可用的 Kotlin 参考。

如下这些例子展示了使用 Kotlin 和 Java 来调用相同的 Android API 的差异：

### 声明一个 Activity

Kotlin

```kotlin
class MyActivity : AppCompatActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity)
  }
}
```

Java

```java
public class MyActivity extends AppCompatActivity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity);
  }
}
```

### 创建一个 `OnClickListener`

Kotlin

```kotlin
val fab = findViewById(R.id.fab) as FloatingActionButton
fab.setOnClickListener {
  ...
}
```

Java

```java
FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
fab.setOnClickListener(new View.OnClickListener() {
  @Override
  public void onClick(View view) {
    ...
  }
});
```

### 创建一个 `ItemSelectedListener`

Kotlin

```kotlin
private val mOnNavigationItemSelectedListener
    = BottomNavigationView.OnNavigationItemSelectedListener { item ->
  when (item.itemId) {
    R.id.navigation_home -> {
      mTextMessage.setText(R.string.title_home)
      return@OnNavigationItemSelectedListener true
    }
    R.id.navigation_dashboard -> {
      mTextMessage.setText(R.string.title_dashboard)
      return@OnNavigationItemSelectedListener true
    }
 }
 false
}
```

Java

```java
private BottomNavigationView.OnNavigationItemSelectedListener mOnNavigationItemSelectedListener
    = new BottomNavigationView.OnNavigationItemSelectedListener() {
  @Override
  public boolean onNavigationItemSelected(@NonNull MenuItem item) {
    switch (item.getItemId()) {
      case R.id.navigation_home:
        mTextMessage.setText(R.string.title_home);
        return true;
      case R.id.navigation_dashboard:
        mTextMessage.setText(R.string.title_dashboard);
        return true;
    }
    return false;
  }
};
```


## 最佳实践

当您逐渐熟悉 Kotlin，请试着遵循如下的准则：

* 不要过分准求精简代码行数而破坏了可读性。在 Kotlin 提供了众多语法糖（syntactic sugar）的情况下，很容易做得过火。
* 最好确立让整个团队最适应的代码规范和风格。[Kotlin](http://kotlinlang.org/docs/reference/coding-conventions.html) 和 [Android Kotlin](https://android.github.io/kotlin-guides/) 代码风格指南里有不少格式化 Kotlin 代码的靠谱建议。

