# Kotlin



## 1. 变量声明

- **`fun`** 代表函数。函数是程序中用于执行特定任务的部分。

- **`main`** 是这运行程序时调用的第一个函数（即主函数）。每个 Kotlin 程序都需要有一个 `main` 函数。

- **`println`** 指示系统输出一行文本。

- **`val`** 用于值从不更改的变量。使用 `val` 声明的变量无法重新赋值，`value`缩写。

- **`var`** 用于值可以更改的变量，`variable`缩写。

- **`Int`、`Byte`、`Short`、`Long`、`Float` 和 `Double`**数值类型。

```kotlin
var count: Int = 10
val languageName: String = "Kotlin"
```



### 1.1 类型推断

- Kotlin 是一种静态类型的语言。类型将在编译时解析且从不改变。



### 1.2 `Null`安全

- 默认情况下，Kotlin 变量不能持有 null 值。
- 要使变量持有 null 值，它必须是可为 null 类型。可以在变量类型后面加上 `?`后缀，将变量指定为可为 null。

```kotlin
val languageName: String? = null
```



## 2. 条件语句

- 每个条件分支可以隐式地返回其最后一行的表达式的结果，因此无需使用 `return` 关键字。

```kotlin
val answerString: String = if (count == 42) {
    "I have the answer."
} else if (count > 35) {
    "The answer is close."
} else {
    "The answer eludes me."
}

println(answerString)
```

- 可以将` if-else `表达式替换为 `when` 表达式。`when` 表达式中每个分支都由一个条件、一个箭头 (`->`) 和一个结果来表示。如果箭头左侧的条件求值为 true，则会返回右侧的表达式结果。

```kotlin
val answerString = when {
    count == 42 -> "I have the answer."
    count > 35 -> "The answer is close."
    else -> "The answer eludes me."
}

println(answerString)
```



## 3. 函数



### 3.1 简化函数声明

将 return 关键字替换为赋值运算符。（略丑，不用）



### 3.2 匿名函数

```kotlin
val stringLengthFunc: (String) -> Int = { input ->
    input.length
}
```



### 3.1 高阶函数

> 将其他函数用作参数的函数称为“高阶函数”。（与Java中使用回调接口相同）

```kotlin
fun stringMapper(str: String, mapper: (String) -> Int): Int {
    // Invoke function
    return mapper(str)
}
```



## 4. 类

> 使用 `class` 关键字来定义类。



## 5. 互操作性

Kotlin 代码可编译为 JVM 字节码，因此 Kotlin 代码可直接调用 Java 代码。



# 在 Android 开发中使用常见的 Kotlin 模式



## 1. 基础功能



### 1.1 继承

> 在子类与其父类之间使用 `:` 运算符来指明其继承关系。

```kotlin
class LoginFragment : Fragment()
```

- 使用 `override` 关键字替换函数。
- 使用 `super` 关键字引用父类中的函数。

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
}
```



### 1.2 延迟初始化

可以使用 `lateinit` 推迟属性初始化。

```kotlin
class LoginFragment : Fragment() {

    private lateinit var usernameEditText: EditText
    private lateinit var passwordEditText: EditText
    private lateinit var loginButton: Button
    private lateinit var statusTextView: TextView

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        usernameEditText = view.findViewById(R.id.username_edit_text)
        passwordEditText = view.findViewById(R.id.password_edit_text)
        loginButton = view.findViewById(R.id.login_button)
        statusTextView = view.findViewById(R.id.status_text_view)
    }

    ...
}
```



### 1.3 SAM转换

通过实现 `OnClickListener` 接口来监听 Android 中的点击事件。`Button` 对象包含一个 `setOnClickListener()` 函数，该函数接受 `OnClickListener` 的实现。

```kotlin
loginButton.setOnClickListener {
    val authSuccessful: Boolean = viewModel.authenticate(
            usernameEditText.text.toString(),
            passwordEditText.text.toString()
    )
    if (authSuccessful) {
        // Navigate to next screen
    } else {
        statusTextView.text = requireContext().getString(R.string.auth_failed)
    }
}
```

待深入理解。



### 1.4 伴生对象

用于定义在概念上与某个类型相关但不与某个特定对象关联的变量或函数。伴生对象类似于对变量和方法使用 Java 的 `static`关键字。

```kotlin
class LoginFragment : Fragment() {

    ...

    companion object {
        private const val TAG = "LoginFragment"
    }
}
```



### 1.5 属性委托

属性委托提供了一种可在整个应用中重复使用的通用实现。

```kotlin
private val viewModel: LoginViewModel by viewModels()
```



## 2. 可为`null`性

> 对象的引用不能包含 null 值。如需为变量赋 null 值，必须通过将 `?` 添加到基本类型的末尾以声明可为 null 变量类型。

```kotlin
val name: String? = null
```

可为 null 性是 Java 和 Kotlin 在行为上有所不同的一个主要方面。Java 对可为 null 性语法的要求不那么严格。



### 2.1 平台类型

- 编译器将不知道 `String` 映射到 Kotlin 中的 `String` 还是 `String?`。这种不明确性通过平台类型 `String!` 表示。
- 当用 Java 编写代码时，都应使用可为 null 性注释。

```java
public class Account implements Parcelable {
    public final String name;
    public final String type;
    private final @Nullable String accessId;

    ...
}
```

```java
public class Account implements Parcelable {
    public final @NonNull String name;
    ...
}
```



### 2.2 处理可为`null	`性

- 使用非 null 断言运算符 `!!`，如果它左侧表达式的结果为 null，则您的应用会抛出 `NullPointerException`，应谨慎使用。

```kotlin
val account = Account("name", "type")
val accountName = account.name!!.trim()
```

- 使用安全调用运算符 `?.`，安全调用运算符可避免潜在的 `NullPointerException`，但它会将 null 值传递给下一个语句。可以使用 Elvis 运算符 (`?:`) 紧接着处理 null 值的情况。

```kotlin
val account = Account("name", "type")
val accountName = account.name?.trim() ?: "Default name"
```



### 2.3 属性初始话

- 类声明中初始化；
- 初始化式块中定义；
- 初始化使用 `lateinit`；



### 2.4 内容提供程序

> *内容提供程序*管理一组共享的应用数据，您可以将这些数据存储在文件系统、SQLite 数据库、网络中或者您的应用可访问的任何其他持久化存储位置。其他应用可通过内容提供程序查询或修改数据（如果内容提供程序允许）。

- 分配 URI 无需应用保持运行状态，因此 URI 可在其所属的应用退出后继续保留。当系统必须从相应的 URI 检索应用数据时，系统只需确保所属应用仍处于运行状态。
- 这些 URI 还会提供重要的细粒度安全模型。例如，应用可将其所拥有图像的 URI 放到剪贴板上，但将其内容提供程序锁定，以便其他应用程序无法随意访问它。当第二个应用尝试访问剪贴板上的 URI 时，系统可允许该应用通过临时的 *URI 授权*来访问数据，这样便只能访问 URI 后面的数据，而非第二个应用中的其他任何内容。



# Kotlin样式



## 1. 源文件

> 所有源文件都必须编码为 UTF-8。

- 文件名应为顶级类的名称（区分大小写）加上 `.kt` 扩展名。
- 制表符不用于缩进。
- 对于任何具有特殊转义序列（`\b`、`\n`、`\r`、`\t`、`\'`、`\"`、`\\` 和 `\$`）的字符，将使用该序列，而不是相应的 Unicode 转义字符（例如 `\u000a`）。
- 建议不要对任何位置的可打印字符使用 Unicode 转义字符，强烈建议不要在字符串字面量和注释之外使用 Unicode 转义字符

`.kt` 文件由下面几部分组成（按顺序列出）：

- 版权和/或许可标头（可选）
- 文件级注解
- package 语句
- import 语句
- 顶级声明

上述各部分用一个空白行隔开。



## 2. 格式规范

> 和Java编程规范基本一致，此处仅列举不一致的地方。

- 软件包名称全部为小写字母，连续的单词直接连接在一起（没有下划线）。
