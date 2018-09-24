# Kotlin 和 Android
> 原文链接：[Kotlin and Android  |  Android Developers](https://developer.android.google.cn/kotlin/)

现在，您可以用干练、强大、富有表达力的 Kotlin 来开发 Android 应用。而且，最棒的是它可以和 Java 编程语言、以及现有的 Android 运行时框架互用（interoperable）。

#### [开始动手](https://developer.android.google.cn/kotlin/get-started)

## 现代、安全、表达力强

Kotlin 既精简干练又富有表达力。它含有针对空值（nullability）和不可变性（immutability）的安全特性，让您的应用从一开始就保持健康高效。

## 更安全的代码

写更安全的代码，在您的应用中避免 `NullPointerException`。

```kotlin
var output: String
output = null   // 编译错误

==================================

val name: String? = null    // 可为空值的类型
println(name.length())      // 编译错误
```

## 精简干练、可读性高

### 数据类（Data class）

集中注意力在您的点子上，少写八股代码（boilerplate code）。

```kotlin
// 这一行代码就能创建一个含有 getter、setter、equals()、
// hashCode()、toString() 和 copy() 的 POJO:
data class User(val name: String, val email: String)
```

### 匿名函数（Lambda）

使用匿名函数来简化您的代码。

```kotlin
button.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v){
      doSomething();
    }
});
```

```kotlin
button.setOnClickListener { doSomething() }
```

### 默认实参（default argument）、命名实参（named argument）

用默认实参来减少函数重载。使用命名实参来调用函数，让您的代码更具有可读性。

```kotlin
fun format(str: String,
    normalizeCase: Boolean = true,
    upperCaseFirstLetter: Boolean = true,
    divideByCamelHumps: Boolean = false,
    wordSeparator: Char = ' ') {
    …
}

==================================

// 使用命名实参来调用函数
format(str, normalizeCase = true, upperCaseFirstLetter = true)
```

## 和 **findViewById** 说再见

在您的代码中避免啰嗦的 `findViewById()` 调用，从而把精力集中在编写业务逻辑上。

```kotlin
import kotlinx.android.synthetic.main.content_main.*

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // 毋须调用 findViewById(R.id.textView) 来返回一个 TextView
        textView.text = "Kotlin for Android 嘹咋咧！"
    }
}
```

## 不继承父类却仍能拓展功能

扩展函数（Extension function）和扩展属性（Extension properties）让您在不继承类的情况下也能轻易地扩展它的功能，而且，这种扩展代码的调用有着良好的可读性。

```kotlin
// 使用 inflate 函数来扩展 ViewGroup
fun ViewGroup.inflate(layoutRes: Int): View {
    return LayoutInflater.from(context).inflate(layoutRes, this, false)
}

==================================

// 在 ViewGroup 的实例上直接调用 inflate
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
    val v = parent.inflate(R.layout.view_item)
    return ViewHolder(v)
}
```

## 和 Java 的完全互用性

您可以为 Java 项目中随意添加或少或多 Kotlin 代码，因为它是一个和 Java 编程语言可以完全互用的 JVM 语言。

```kotlin
// 在 Kotlin 中调用 Java 代码
class KotlinClass {
    fun kotlinDoSomething() {
        val javaClass = JavaClass()
        javaClass.javaDoSomething()
        println(JavaClass().prop)
    }
}

==================================

// 在 Java 中调用 Kotlin 代码
public class JavaClass {
    public String getProp() { return "Hello"; }
    public void javaDoSomething() {
        new KotlinClass().kotlinDoSomething();
    }
}
```

## 出色的工具支持

Android Studio 3.0 提供了帮助您开始使用 Kotlin 的工具，让您能把整个 Java 文件转换成 Kotlin，或者在向 Kotlin 文件拷贝 Java 代码片段的时候随手转换。

![](https://developer.android.google.cn/images/kotlin/kotlin-convert_2x.png)

## 开源的 Kotlin

正如 Android 那样，Kotlin 是按照 Apache 2.0 协议开源的项目。在 Android 平台演化进步的时候，我们对 Kotlin 的选择再一次证实了我们对于开源社区生态的承诺，这个语言的日益精进更是令我们亦可赛艇。

#### [在 GitHub 上查看 Kotlin 项目](https://github.com/JetBrains/kotlin)

## 使用 Kotlin 编写的应用

许多应用已经使用 Kotlin 编写，其中不乏最炙手可热的创业公司和财富五百强的一本大厂。

