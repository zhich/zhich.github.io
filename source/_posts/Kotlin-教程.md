---
title: Kotlin 教程
date: 2023-04-26 21:47:00
categories: "Android"
tags:
     - Android
     - Kotlin
---

<meta name="referrer" content="no-referrer" />





### 基础类型

Java 中存在 int, float, boolean 等基础类型，这些基础类型在 Kotlin 里将全部以对象的形式继续存在。有几点变化需要注意。

```Kotlin
// Int 无法自动转换为 Double, 需要自己先做类型转换（as Double, toDouble(), 方式很多）
var a: Int = 2
var b: Double = a.toDouble()

// Char 不能直接等值于其对应的 ASCII 码值，需要类型转换
var c: Char = 'a'
var x: Int = c.toInt()
```

### 变量声明

Kotlin 中使用 var 定义可读可写变量，使用 val 定义只读变量（相当于 Java 当中的 final）。定义变量时，如果满足类型推断，类型可以省略。

```Kotlin
var id = "100" // 类型为 String
var name: String = "张三" // 类型为 String
val age: Int = 30
```

### 类型声明

在 Kotlin 语言中，“:” 被广泛用于变量类型的定义。

```Kotlin
// 定义变量类型
fun test() {
    var zhangSan: User
    var liSi: User
}

// 定义函数参数和返回值
fun getUser(id: Int): User {
    return User(100, "dd", 30)
}
```

“:”还被用于声明类继承或接口实现。

```Kotlin
open class User {
    constructor()
}

// 继承 User 类
class VipUser : User() {

}

interface DB {
    fun addUser(user: User)
}

// 实现 DB 接口
class RoomDB : DB {
    override fun addUser(user: User) {

    }
}
```

### 类型检测

Java 中使用 instanceof 来判断某变量是否为某类型，而 Kotlin 中使用 is 来进行类型检测。

```Kotlin
/**
 * is 判断属于某类型
 * !is 判断不属于某类型
 * as 类型强转，失败时抛出类型强转失败异常
 * as? 类型强转，但失败时不会抛出异常而是返回 null
 */
if (myObject is User) {
    // 只要进来了，就代表 myObject 是 User 类型了，不需要强转就可以直接使用 User 的属性
    myObject.id = 101
    myObject.name = "Hello"
}
```

### Any 和 Unit

- **Any**: Kotlin 的顶层父类是 Any, 对应 Java 中的 Object, 但是比 Object 少了 wait()/notify() 等函数。
- **Unit**: Kotlin 中的 Unit 对应 Java 中的 void。 

### 可见性修饰符

- 默认的可见性修饰符是 `public`。
- 新增的可见性修饰符 `internal` 表示仅当前模块（AS 中的 module）可见，其它模块不能访问。

### 逻辑语句

#### if-else 语句

Kotlin 中的 if-else 基本和 Java 一致，但还是有一些特殊的地方。比如它可以作为一个逻辑表达式使用，逻辑表达式还可以以代码块的形式出现，代码块最后的表达式作为该块的返回值。

```Kotlin
// 逻辑表达式的使用
fun getMax(x: Int, y: Int): Int {
    var max = if (x > y) x else y
    return max
}

// 代码块形式的逻辑表达式
fun getMax2(x: Int, y: Int): Int {
    var max = if (x > y) {
        println("Max num is x.")
        x // 返回最后一行，即 x 的值
    } else {
        println("Max num is y.")
        y // 返回最后一行，即 y 的值
    }
    return max
}
```

#### when 语句

Kotlin 中的 when 语句取代了 Java 中的 switch-case 语句，功能上要强大许多，可以有多种形式的条件表达。与 if-else 一样，Kotlin 中的 when 也可以作为逻辑表达式使用，也有返回值。

```Kotlin
// when 有判断参数
fun whenDemo(obj: Any) {
    when (obj) {
        1 -> println("是数字 1")
        -1, 0 -> println("是数字 -1 或 0")
        in 1..10 -> println("是不大于 10 的正整数")
        "abc" -> println("是字符串 abc")
        is User -> {
            println("是 User 对象")
            println(obj.name) // 直接可以使用 User 的属性了
        }
        else -> println("其它操作")
    }
}

// when 没有判断参数
fun whenDemo2(position: Int) {
    val columns = 10
    when {
        position % columns == 0 -> { // position 位于第一列
            // ...
        }
        (position + 1) % columns == 0 -> { // position 位于最后一列
            // ...
        }
    }
}

// when 有返回值
fun whenDemo3(score: Int) {
    val result = when (score) {
        in 90..100 -> "优秀"
        in 80..89 -> "良好"
        in 60..79 -> "及格"
        else -> "不及格"
    }
    println("你的成绩$result")
}
```

#### 循环

```Kotlin
// 标准函数 repeat()：
repeat(100) { // 执行 100 次
}

// .. 包括右边界 [0,99]
for (i in 0..99) {
}
// until 不包括右边界 [0,99]
for (i in 0 until 100) {
}
```

> while 语句、continue 语句和 break 语句等逻辑都与 Java 基本一致，这里不再赘述。

#### 标签

Kotlin 中可以对任意表达式进行标签标记，标签的格式为标识符后跟 @ 符号，例如 abc@、fooBar@ 都是有效的标签。这些标签，可以搭配 return、break、continue 等跳转行为来使用。

```Kotlin
fun labelTest() {
    la@ for (i in 1..10) {
        println("outer index $i")
        for (j in 1..10) {
            println("inner index $j")
            if (j % 2 == 0) {
                break@la
            }
        }
    }
}
```

### 数组

使用 `arrayOf` 创建数组，基本数据类型使用对应的 `intArrayOf` 等。

```Kotlin
val arr1 = arrayOf<Int>()
var arr2 = intArrayOf()
var arr3 = floatArrayOf()
```

### 字符串模板

```Kotlin
// Java 中字符串模板
String name = "ZhangSan";
int age = 30;
String introduction = String.format("我是s%，今年%d岁了", name, age);

// Kotlin 中字符串模板
val name = "ZhangSan"
val age = 30
val introduction = "我是${name}，今年${age}岁了"
```

### 函数

Kotlin 中的函数通过关键字 fun 定义的，具体的参数和返回值定义结构如下。

```Kotlin
fun myFun(para1: Int, para2: String): String { ... }
```

Kotlin 中的函数可以是全局函数，成员函数或者局部函数，甚至还可以作为某个对象的扩展函数临时添加。

#### 全局函数/顶级函数

全局函数（顶级函数）是文件级别的，以包为作用域。

例如 com.zch.kotlin.biz1 包中的 Common.kt 文件中有以下方法：

```Kotlin
fun sayHello() {
    println("Hello")
}
```

那么在当前包的其它文件中就不能有相同的方法 sayHello，否则会报错。

如果 com.zch.kotlin.biz2 包中的某文件中也有以下方法：

```Kotlin
fun sayHello() {
    println("Hello")
}
```

那么在其它文件中同时调用两者就需要加上相应的包来访问。

```Kotlin
import com.zch.kotlin.biz1.sayHello

fun hello() {
    sayHello()
    com.zch.kotlin.biz2.sayHello()
}
```

#### 成员函数

成员函数是在类或对象内部定义的函数。

#### 局部函数

Kotlin 支持局部函数，即一个函数在另一个函数内部：

```Kotlin
private fun getUserInfo(): String {
    val country = "中国"

    // 函数中的函数，叫“局部函数”
    fun getProvince(): String {
        return "${country}广东省" // 局部函数可以访问外部函数的局部变量。
    }

    val name = "ZhangSan"
    val age = 30
    val userInfo = "我是${name}，今年${age}岁了，我来自${getProvince()}。"

    return userInfo
}
```

> 如果是频繁调用的函数，不建议声明为局部函数，因为每次调用时，就会产生一个函数对象。

#### 函数参数默认值

函数中的某个参数可以用 “=” 号指定其默认值，调用函数方法时可不传这个参数，但其它参数需要用 “=” 号指定。

```Kotlin
fun calculate(a: Int, b: Int = 10, c: Int) = a + b + c // 原本直接 return 的函数可以用 = 符号简化

val sum = calculate(5, c = 20) // sum = 35
```

#### 扩展函数和扩展属性

Kotlin 支持在**包范围内**对已存在的类进行函数和属性扩展。

```Kotlin
// 给 Activity 扩展一个 log 函数
fun Activity.log(msg: String) {
    Log.d("tag", "Activity---$msg")
}

// 给 Context 扩展一个 log 函数
fun Context.log(msg: String) {
    Log.d("tag", "Context---$msg")
}

log("hello") // 调用 Activity.log
(this as Context).log("hello2") // 调用 Context.log

// 给 ViewGroup 扩展一个 firstChild 属性
val ViewGroup.firstChild: View get() = getChildAt(0)

contentLayout.firstChild // 调用
```

> 注意：1、扩展需要在包级范围内进行，如果写在 class 内是无效的。2、已经存在的方法或属性，是无法被扩展的，依旧会调用已有的方法。3、扩展函数是静态解析的，在编译时就确定了调用函数（没有多态）。

#### infix 函数

标有 `infix` 关键字的函数也可以使用中缀表示法（忽略该调用的点与圆括号）调用。中缀函数必须满足以下要求：

- 它们必须是成员函数或扩展函数；
- 它们必须只有一个参数；
- 其参数不得接受可变数量的参数且不能有默认值。

```Kotlin
infix fun Int.shl(x: Int): Int { …… }

// 用中缀表示法调用该函数
1 shl 2

// 等同于这样
1.shl(2)
```

```Kotlin
// 中缀函数调用的优先级低于算术操作符、类型转换以及 rangeTo 操作符。 以下表达式是等价的：

1 shl 2 + 3 等价于 1 shl (2 + 3)
0 until n * 2 等价于 0 until (n * 2)
xs union ys as Set<*> 等价于 xs union (ys as Set<*>)

// 另一方面，中缀函数调用的优先级高于布尔操作符 && 与 ||、is- 与 in- 检测以及其他一些操作符。这些表达式也是等价的：

a && b xor c 等价于 a && (b xor c)
a xor b in c 等价于 (a xor b) in c
```

中缀函数总是要求指定接收者与参数。当使用中缀表示法在当前接收者上调用方法时，需要显式使用 `this`；不能像常规方法调用那样省略。这是确保非模糊解析所必需的。

```Kotlin
class MyStringCollection {
    infix fun add(s: String) { /*……*/ }

    fun build() {
        this add "abc"   // 正确
        add("abc")       // 正确
        add "abc"      // 错误：必须指定接收者
    }
}
```

#### 内联函数

- 内联函数配合「函数类型」，可以减少「函数类型」生成的对象  。
- 使⽤ `inline` 关键字声明的函数是「内联函数」，在「编译时」会将「内联函数」中的函数体直接插⼊到调⽤处。  所以在写内联函数的时候需要注意，尽量将内联函数中的代码行数减少。

`noinline` 可以禁止部分函数参数参与内联编译：

```Kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined:() -> Unit) {
    //......
}
```

#### 匿名函数

- 匿名函数的特点是可以明确指定其返回值类型。
- 它和常规函数的定义几乎相似。他们的区别在于，匿名函数没有函数名。

```Kotlin
// 常规函数
fun test(x : Int , y : Int) : Int = x + y

// 匿名函数
fun(x : Int , y : Int) : Int = x + y
```

#### Lambda 表达式

`Lambda` 表达式的本质是 `匿名函数`，因为在其底层实现中还是通过匿名函数来实现的。但是我们在用的时候不必关心其底层实现。不过 Lambda 的出现确实是减少了代码量的编写，同时也使代码变得更加简洁明了。

**1、Lambda 的特点：**

- Lambda 表达式总是被大括号括着。
- 其参数（如果存在）在符号 “->” 之前声明（参数类型可以省略）。
- 函数体（如果存在）在符号 “->” 后面。

**2、Lambda 语法：**

```Kotlin
1、无参数的情况：
val/var 变量名 = { 操作的代码 }

// 源代码
private fun myFun1() {
    println("无参数")
}

// lambda 代码
val myFun1 = { println("无参数") }

private fun test1() {
    myFun1() // 调用
}
// -------------------------------------------
2、有参数的情况：
val/var 变量名 : (参数的类型，参数类型，...) -> 返回值类型 = {参数1，参数2，... -> 操作参数的代码 }
可等价于
// 此种写法：即表达式的返回值类型会根据操作的代码自推导出来。
val/var 变量名 = { 参数1 ： 类型，参数2 : 类型, ... -> 操作参数的代码 }

// 源代码
fun myFun2(a: Int, b: Int): Int {
    return a + b
}

// lambda
val myFun2: (Int, Int) -> Int = { a, b -> a + b }
// 或者
val myFun2 = { a: Int, b: Int -> a + b }

fun test2() {
    myFun2(1, 2) // 调用
}
// -------------------------------------------
3、lambda 表达式作为函数中的参数：
fun test(a : Int, 参数名 : (参数1 ： 类型，参数2 : 类型, ... ) -> 表达式返回类型){ ... }

// 源代码
fun myFun3(a: Int, b: Int): Int {
    return a + b
}

fun sum(num1: Int, num2: Int): Int {
    return num1 + num2
}

// lambda
fun myFun33(a: Int, b: (num1: Int, num2: Int) -> Int): Int {
    return a + b.invoke(3, 5)
}

fun test3() {
    myFun33(10, { num1: Int, num2: Int -> num1 + num2 }) // 调用
}
```

> invoke() 函数：表示为通过函数变量调用自身，因为上面例子中的变量 b 是一个匿名函数。

**3、it**

当一个高阶函数中 Lambda 表达式的参数只有一个的时候可以使用 it 来使用此参数。it 可表示为单个参数的隐式名称，是 Kotlin 语言约定的。

```Kotlin
val arr = intArrayOf(1, 2, 3)
var sum = 0
arr.forEach {
    sum += it
}
```

**4、下划线（_）**

在使用 Lambda 表达式的时候，可以用下划线 （_） 表示未使用的参数，表示不处理这个参数。

```Kotlin
val map = mapOf("key1" to "value1", "key2" to "value2", "key3" to "value3")
// 不需要 key 的时候
map.forEach { _, value ->
    println("$value")
}
```

**5、带接收者的函数字面值**

在 Kotlin 中，提供了指定的接受者对象调用 Lambda 表达式的功能。在函数字面值的函数体中，可以调用该接收者对象上的方法而无需任何额外的限定符。它类似于扩展函数，它允许在函数体内访问接收者对象的成员。

**5.1、匿名函数作为接收者类型**

匿名函数语法允许你直接指定函数字面值的接收者类型，如果你需要使用带接收者的函数类型声明一个变量，并在之后使用它，这将非常有用。

```Kotlin
val add = fun Int.( other : Int) : Int = this + other
println(2.add(3)) // 输出 5
```

**5.2、Lambda 表达式作为接收者类型**

要用 Lambda 表达式作为接收者类型的前提是**接收者类型可以从上下文中推断出来**。

```Kotlin
class HTML {
    fun body() {}
}

fun myFun5(init: HTML.() -> Unit): HTML {
    val html = HTML() // 创建接收者对象
    html.init()       // 将该接收者对象传给该 lambda
    return html
}

fun test111() {
    myFun5 { // 带接收者的 lambda 由此开始
        body() // 调用该接收者对象的一个方法
    }
}
```

**6、闭包**

所谓闭包，即是函数中包含函数，这里的函数我们可以包含（Lambda 表达式，匿名函数，局部函数，对象表达式）。我们熟知，函数式编程是现在和未来良好的一种编程趋势，故而 Kotlin 也有这一个特性。Java 是不支持闭包的。

**6.1、携带状态**

```Kotlin
// 让函数返回一个函数，并携带状态值
fun myFun4(b: Int): () -> Int {
    var a = 3
    return fun(): Int {
        a++
        return a + b
    }
}

fun test4() {
    val t = myFun4(3)
    println(t()) // 输出 7
    println(t()) // 输出 8
    println(t()) // 输出 9
}
```

**6.2、引用外部变量，并改变外部变量的值**

```Kotlin
var sum : Int = 0
val arr = arrayOf(1,3,5,7,9)
arr.filter { it < 7  }.forEach { sum += it }

println(sum) // 输出 9
```

**7、Lambda 表达式简写**

```Kotlin
// 如果函数的最后⼀个参数是 lambda, 那么 lambda 表达式可以放在圆括号之外：
lessons.forEach(){ lesson : Lesson ->
   // ...
}

// 如果你的函数传⼊参数只有⼀个 lambda 的话，那么⼩括号可以省略的：
lessons.forEach { lesson : Lesson ->
   // ...
}

// 如果 lambda 表达式只有⼀个参数，那么可以省略，通过隐式的 it 来访问：
lessons.forEach { // it
   // ...
}
```

#### 高阶函数

在 `Kotlin` 中，高阶函数即指：将函数用作一个函数的参数或者返回值的函数。

**将函数用作函数参数的情况**

```Kotlin
// sumBy 函数的源码
public inline fun CharSequence.sumBy(selector: (Char) -> Int): Int {
    var sum: Int = 0
    for (element in this) {
        sum += selector(element)
    }
    return sum
}

// 调用
fun test() {
    val str = "abc"
    val sum = str.sumBy { it.toInt() }
    println(sum) // 输出 294。因为字符 a 对应的值为 97，b 对应 98，c 对应 99。故而该值即为 97 + 98 + 99 = 294
}
```

**将函数用作一个函数的返回值**

```Kotlin
fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    } finally {
        lock.unlock()
    }
}

// 调用
fun toBeSynchronized() = sharedResource.operation()
val result = lock(lock, ::toBeSynchronized) 

// 上面的写法也可以写作：
val result = lock(lock, {sharedResource.operation()} )
// 或者
val result = lock(lock) {
    sharedResource.operation()
}
```

> ::toBeSynchronized 即为对函数 toBeSynchronized() 的引用。

**自定义高阶函数**

```Kotlin
// 定义高阶函数
private fun calculation(num1: Int, num2: Int, result: (Int, Int) -> Int): Int {
    return result(num1, num2)
}

// 测试调用
fun test() {
    val result1 = calculation(1, 2) { num1, num2 ->
        num1 + num2
    }
    val result2 = calculation(10, 20) { num1, num2 ->
        num1 * num2
    }
    println("result1 = $result1") // 输出：3
    println("result2 = $result2") // 输出：200
}
```

**开发中常用的一个例子**

```Kotlin
class ParamView @JvmOverloads constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0) : LinearLayout(context, attrs, defStyleAttr) {

    var onParamValueClickListener: (() -> Unit)? = null // 定义一个点击事件，给外部处理

    init {
        tvParamValue?.setOnClickListener {
            onParamValueClickListener?.invoke()
        }
    }
}

// 外部 ParamView 控件处理点击事件
paramView?.onParamValueClickListener = {
    // 处理业务逻辑...
}
```

#### 标准高阶函数

Standard.kt 文件中提供了一系列标准的高阶函数。

**函数特点汇总**

| 函数        | 接收者（this） | 传参（it） | 返回值（result） |
|:--------- |:--------- |:------ |:----------- |
| run()     | 当前类       | 编译错误   | 作用域中的最后一个对象 |
| T.run()   | 类T        | 编译错误   | 作用域中的最后一个对象 |
| with()    | 类T        | 编译错误   | 作用域中的最后一个对象 |
| T.let()   | 当前类       | 类T     | 作用域中的最后一个对象 |
| T.also()  | 当前类       | 类T     | 类T          |
| T.apply() | 类T        | 编译错误   | 类T          |

实际应用：

```Kotlin
class MyStandard {
    companion object {
        const val TAG = "MyStandard"
    }

    // run()
    fun runDemo1() {
        var name = "AA"
        run {
            val name = "BB"
            Log.e(TAG, name) // BB
        }
        Log.d(TAG, name) // AA
    }

    // T.run()
    fun runDemo2() {
        val result = "ABCDEF".run {
            Log.d(TAG, "字符串的长度为 $length") // 字符串的长度为 6
            substring(2)
        }
        Log.d(TAG, result) // CDEF
    }

    // with()
    fun withDemo() {
        val result = with("ABCDEF") {
            substring(2)
        }
        Log.d(TAG, result) // CDEF
    }

    // T.let()
    fun letDemo() {
        val result = "ABCDEF".let {
            it.substring(2) // it 代表 "ABCDEF"
        }
        Log.d(TAG, result) // CDEF
    }

    // T.also()
    fun alsoDemo() {
        val result = "ABCDEF".also {
            it.substring(2) // it 代表 "ABCDEF"
        }
        Log.d(TAG, result) // ABCDEF
    }

    // T.apply()
    fun applyDemo() {
        val result = "ABCDEF".apply {
            this.substring(2) // this 代表 "ABCDEF"
        }
        Log.d(TAG, result) // ABCDEF
    }
}
```

通常，T.run()、T.let()、T.also() 和 T.apply() 四个用的比较多，使用时可以通过简单的规则作出⼀些判断：

```Kotlin
if(需要返回自身){
    if(作用域中使用 this 作为参数){
        选择 apply
    }
    if(作用域中使用 it 作为参数){
        选择 also
    }
}
if(不需要返回自身){
    if(作用域中使用 this 作为参数){
        选择 run
    }
    if(作用域中使用 it 作为参数){
        选择 let
    }
}
```

### 实化类型参数

**泛型类型擦除**：JVM 中的泛型一般是通过类型擦除实现的，也就是说泛型类实例的类型实参在编译时被擦除，在运行时是不会被保留的。基于这样实现的做法是有历史原因的，最大的原因之一是为了兼容 JDK1.5 之前的版本，当然泛型类型擦除也是有好处的，在运行时丢弃了一些类型实参的信息，对于内存占用也会减少很多。

正因为泛型类型擦除原因在业界 Java 的泛型又称伪泛型。因为编译后所有泛型的类型实参类型都会被替换成 Object 类型或者泛型类型形参指定上界约束类的类型。例如： List<Float>、List<String>、List<Student> 在 JVM 运行时 Float、String、Student 都被替换成 Object 类型，如果泛型定义是 List<T extends Student> 那么运行时 T 被替换成 Student 类型。

Kotlin 和 Java 都存在泛型类型擦除的问题，但 Kotlin 可以通过 `inline` 函数保证使得泛型类的类型实参在运行时能够保留，这样的操作 Kotlin 中把它称为**实化**，对应需要使用 **reified** 关键字。因此，我们可以通过配合 `inline + reified` 达到「真泛型」的效果。

**1、满足实化类型参数函数的必要条件**

- 必须是 inline 内联函数，使用 inline 关键字修饰。
- 泛型类定义泛型形参时必须使用 reified 关键字修饰。

**2、带实化类型参数的函数基本定义**

```Kotlin
// 定义
inline fun <reified T> isInstanceOf(value: Any): Boolean = value is T

val obj = User()
val instanceOf = isInstanceOf<User>(obj) // 调用
```

对于以上例子，我们可以说类型形参 T 是泛型函数 isInstanceOf 的实化类型参数。

**3、原理描述**

编译器把实现内联函数的字节码动态插入到每次的调用点。那么实化的原理正是基于这个机制，**每次调用带实化类型参数的函数时，编译器都知道此次调用中作为泛型类型实参的具体类型。所以编译器只要在每次调用时生成对应不同类型实参调用的字节码插入到调用点即可。** 总之一句话很简单，就是带实化参数的函数每次调用都生成不同类型实参的字节码，动态插入到调用点。由于生成的字节码的类型实参引用了具体的类型，而不是类型参数所以不会存在擦除问题。

**4、实化类型参数函数不能在 Java 中调用**

Kotlin 的实化类型参数函数主要得益于 inline 函数的内联功能，虽然 Java 可以调用普通的内联函数但是失去了内联功能，失去内联功能也就意味实化操作也就化为泡影。

**5、实化类型参数函数的使用限制**

- 不能使用非实化类型形参作为类型实参调用带实化类型参数的函数。
- 不能使用实化类型参数创建该类型参数的实例对象。
- 不能调用实化类型参数的伴生对象方法。
- reified 关键字只能标记实化类型参数的内联函数，不能作用于类和属性。

**6、实化类型参数函数使用例子**

```Kotlin
inline fun <reified T> create(): T {
    return RETROFIT.create(T::class.java)
}

val api = create<API>() // 调用

inline fun <reified T> Gson.fromJson(json: String) = fromJson(json, T::class.java)

val user = Gson().fromJson<User>(json) // 调用
```

### 类与继承

Kotlin 中也使用 class 关键字定义类，所有类都继承于 Any 类，类似于 Java 中 Object 类的概念。类实例化的形式也与 Java 一样，但是去掉了 new 关键字。

#### 构造器

类的构造器分为主构造器（primary constructor）和次级构造器（secondary constructor），前者只能有一个，而后者可以有多个。如果两者都未指定，则默认为无参数的主构造器。

主构造器是属于类头的一部分，用 constructor 关键字定义，如果没有被「可见性修饰符」或者「注解」标注，constructor 可省略。由于主构造器不能包含任何代码，初始化代码需要单独写在 init 代码块中，主构造器的参数只能在 init 代码块和变量初始化时使用。

次级构造器也是用 constructor 关键字定义，必须要直接或间接代理主构造器。

```Kotlin
class User(name: String) { // 主构造器
    var id: Int = 0
    var name: String = ""

    init {
        // ...
    }

    constructor(id: Int, name: String) : this(name) { // 次级构造器
        this.id = id
        this.name = name
    }
}
```

主构造器中的参数加上 val 或者 var 修饰，那么参数就变成类的成员变量，如果参数和类原来的成员变量一样，那么对应原来的成员变量就要去掉。因此，以上代码一般可以简化为以下代码。

```Kotlin
class User(val id: Int, val name: String) {
    init {
        // ...
    }
}
```

#### 继承

类继承使用符号 “:” 表示，接口实现也一样，不做原本 Java 中的 extends 和 implement 关键字区分。Kotlin 中取消了 final 关键字，所有类默认都是被 final 修饰，不能被继承。Kotlin 中新增了 open 关键字，被 open 或者 abstract 修饰的类才可以被继承。

#### 单例与伴随对象

Kotlin 使用关键词 object 定义单例类。这里需要注意，是全小写。单例类访问直接使用类名，无构造函数。

```Kotlin
object LogUtil {
    fun d(msg: String) {
        if (BuildConfig.DEBUG) {
            Log.d("tag", msg)
        }
    }
}
```

> 可通过 AS 工具栏 Tools --> Kotlin --> Show Kotlin Bytecode, 点击右侧的 Decompile 来把当前 Kotlin 代码转换为 Java 代码，来验证以上 object 对象是否转换成 Java 中的单例类。

Java 中使用 static 标识一个类里的静态属性或方法。Kotlin 中没有 static 关键字，改为使用伴随对象，用 companion 修饰单例类 object, 来实现静态属性或方法功能。

```Kotlin
class LoginCache {
    companion object {
        const val USER_NAME = "user_name"
        const val PASSWORD = "password"

        fun saveLoginInfo(userName: String, password: String) {

        }
    }
}

LoginCache.saveLoginInfo("ZhangSan","123456") // 调用
```

#### 内部类

在 Kotlin 中，内部类默认是静态内部类，通过 `inner` 关键字声明为嵌套内部类。

#### 匿名内部类

形式：

```Kotlin
object : OnItemCLickListener {
}
```

应用：

```Kotlin
btnLogin.setOnClickListener(object :View.OnClickListener{
    override fun onClick(v: View?) {

    }
})

// 以上匿名内部类转换成的 lambda 表达式如下：
btnLogin.setOnClickListener {

}
```

### const

- const 必须修饰 val。
- const 只允许在 top-level 级别和 object 中声明。

```Kotlin
object LogUtil {
    const val TAG: String = "LogUtil"
    val msg: String = "msg"
}
```

以上 Kotlin 代码转换成的 Java 代码如下：

```Java
// 省略了部分不大相关代码
public final class LogUtil {
    public static final String TAG = "LogUtil";
    private static final String msg;

    public final String getMsg() {
        return msg;
    }
    // ...
}
```

可以看出，const val 可见性为 public static final, 可以直接访问。val 可见性为 private static final, 并且 val 会生成对应属性的 getter 方法，通过方法调用访问。当定义常量时，出于效率考虑，应该使用 const val 方式，避免频繁函数调用。const 修饰的静态变量又称为**编译器常量**。

### 声明抽象类/接口/枚举/注解

```Kotlin
// 声明抽象类
abstract class
// 声明接⼝
interface
// 声明注解
annotation class
// 声明枚举
enmu class
```

### 受检异常

Kotlin 不需要使用 `try-catch` 强制捕获异常。

### 获取 Class 对象

```Kotlin
使用 [类名::class] 获取的是 Kotlin 的类型 KClass
使用 [类名::class.java] 获取的是 Java 的类型

startActivity(Intent(this, MainActivity::class.java))
```

### setter/getter

在 Kotlin 声明属性的时候（没有使用 private 修饰），会自动生成一个私有属性和一对公开的 setter/getter 方法。在写 setter/getter 的时候使⽤ field 来代替内部的私有属性（防⽌递归栈溢出）。

```Kotlin
class Person {
    var name: String = ""
        get() = field // 默认实现方式，可省略
        set(value) { // 默认实现方式，可省略
            field = value
        }
    var age: Int = 0
        get() = 18 // 获取的值一直都是 18，不会改变
        set(value) {
            field = if (value < 0) 1 else value
        }
}
```

### JvmXxx 相关注解

#### JvmField

通过 `@JvmField` 注解可以让编译器只生成一个 public 的成员属性，不生成对应的 setter/getter 方法。

#### JvmName

顶层函数在文件中定义函数和属性，会直接生成静态的成员，在 Java 中通过「文件名Kt」来访问，同时可以通过 `@file:JvmName` 注解来修改这个类名。注解要写在包名前面才会起作用。

```Kotlin
@file:JvmName("AppCache")

package com.zch.kotlin.biz1

var mToken: String? = ""
fun saveToken(token: String?) {
    mToken = token
}
```

```Java
// java 中访问方式
public void test() {
    AppCacheKt.saveToken("abc123"); // 原来的访问方式
    AppCache.saveToken("abc123"); // 加了 @file:JvmName 注解后的访问方式
}
```

#### JvmStatic

如果将命名对象或伴生对象中定义的函数注解为 `@JvmStatic` ，Kotlin 会为这些函数生成静态方法。

`命名对象中的 @JvmStatic`

```Kotlin
object SizeUtil {
    @JvmStatic
    fun dp2px() {
    }

    fun sp2px() {}
}
```

```Java
// java 方法
public void testSizeUtil() {
    // 之前的访问方式
    SizeUtil.INSTANCE.dp2px();
    SizeUtil.INSTANCE.sp2px();

    // 加了 @JvmStatic 注解后，dp2px 变成了静态方法
    SizeUtil.dp2px();
}
```

`伴生对象中的 @JvmStatic`

```Kotlin
class Util {
    companion object {
        @JvmStatic
        fun sayYes() {
        }

        fun sayNo() {
        }
    }
}
```

```Java
// java 方法
public void testUtil(){
    // 之前的访问方式
    Util.Companion.sayYes();
    Util.Companion.sayNo();

    /**
     * 加了 @JvmStatic 注解后，Util 生成了静态方法 sayYes，
     * 该静态方法直接访问 Companion 的 sayYes()
     */
    Util.sayYes();
}
```

> `@JvmStatic` 注解也可以应用于对象或伴生对象的属性，使其 getter 和 setter 方法成为该对象或包含伴生对象的类的静态成员。

#### JvmOverloads

在 Kotlin 中调用默认参数值的方法或者构造函数是完全没问题的，Java 中调用相应 Kotlin 方法时，是必须输入所有参数的值的，Kotlin 中默认参数我们无法使用。而当加上 `@JvmOverloads`, Kotlin 编译器生成的字节码中有对应的重载方法，我们就可以通过 Java 的重载方式来使用 Kotlin 的代码了，不必要输入所有的参数。

```Kotlin
class ParamView @JvmOverloads constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0) : LinearLayout(context, attrs, defStyleAttr) {

}
```

以上 Kotlin 的自定义控件加上 `@JvmOverloads` 后，相当于 Java 中的以下代码：

```Java
public class ParamView extends LinearLayout {

    public ParamView(Context context) {
        this(context, null);
    }

    public ParamView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ParamView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
}
```

> Kotlin 中构造函数、顶级函数、类中方法，静态方法（@Jvmstatic 修饰）均可采用 `@JvmOverloads` 生成对应重载方法。

### 注解使用处目标

当某个元素可能会包含多种内容（例如构造属性，成员属性），使用注解时可以通过「注解使⽤处⽬标」，让注解对⽬标发⽣作⽤，例如 @file: 、 @get: 、@set: 等。  

```Kotlin
class App : Application() {
    companion object {
        @JvmStatic
        @get:JvmName("instance")
        lateinit var instance: Application
            private set
    }

    override fun onCreate() {
        super.onCreate()
        instance = this
    }
}
```

```Java
App.Companion.getInstance(); // java 中原来调用方式
App.instance(); // java 中现在调用方式
```

### Elvis 操作符

可通过 `?:` 的操作来简化 `if null` 操作。

```Kotlin
// lesson.date 为空时使⽤默认值
val date = lesson.date?: "⽇期待定"
// lesson.state 为空时提前返回函数
val state = lesson.state?: return
// lesson.content 为空时抛出异常
val content = lesson.content ?: throw IllegalArgumentException("content expected")
```

### 空指针安全

Kotlin 中，当我们定义一个变量时，其默认就是非空类型。如果你直接尝试给他赋值为 null, 编译器会直接报错。Kotlin 中将符号 “?” 定义为安全调用操作符。变量类型后面跟 ? 号定义，表明这是一个可空类型。同样的，在调用子属性和函数时，也可以用字符 ? 进行安全调用。Kotlin 的编译器会在写代码时就检查非空情况。

Kotlin 还提供 “!!” 双感叹号操作符来强制调用对象的属性和方法，无视其是否非空。这是一个挺危险的操作符，除非有特殊需求，否则为了远离 NPE, 还是少用为妙。

```Kotlin
var tvContent: TextView // 非空类型
var tvContent: TextView? // 可空类型

!! // 强行调用符
? // 安全调用符
```

```Kotlin
var s1: String = "abc"
s1 = null // 这里编译器会报错

var s2: String? = "abc"
s2 = null // 编译器不会报错

var l1 = s1.length // 可正常编译
var l2 = s2.length // 没有做非空判断，编译器检查报错

if (s2 != null) s2.length // Java 式的判空方案
s2?.length // Kotlin 的安全调用操作符 ?。当 s2 为 null 时，s2?.length 也为 null

s2!!.length // 可能会导致 NPE
```

### lateinit 关键字

```Kotlin
/**
 * 1、lateinit 只能修饰 var 可读可写变量。
 * 2、lateinit 关键字声明的变量类型必须是不可空类型。
 * 3、lateinit 声明的变量不能有初始值。
 * 4、lateinit 声明的变量不能是基本数据类型。
 * 5、在构造器中初始化的属性不需要 lateinit 关键字。
 */
private lateinit var tvContent: TextView
```

### data class

数据类通常都是由多个属性和对应的 getter、setter 组成。当有大量多属性时，不仅这些类会因为大量的 getter 和 setter 方法而行数爆炸，也使整个工程方法数骤增。

Kotlin 中做了这层特性优化，提供了数据类的简单实现。不用再写 getter、setter 方法，这些都由编译器背后去做，你得到的是一个清爽干净的数据类。数据类用 `data class` 声明。

```Kotlin
data class Student(
    var id: Int,
    var name: String,
    var age: Int
)
```

把这个数据类反编译成 Java 代码可知，数据类除了为我们生成了 getter、setter（val 声明的变量不生成 setter 方法）、构造函数外，还有以下方法。

```Kotlin
equals() / hashCode()
toString()
componentN()...
copy()
```

`copy 函数`

```Kotlin
public final Student copy(int id, String name, int age) {
    return new Student(id, name, age);
}
```

当要复制一个对象，只改变一些属性，但其余不变，copy() 就是为此而生。

`componentN 函数-解构声明（Destructuring Declarations）`

```Kotlin
public final int component1() {
    return this.id;
}

public final String component2() {
    return this.name;
}

public final int component3() {
    return this.age;
}
```

编译器为数据类（data class）自动声明 componentN() 函数，可直接用解构声明。

```Kotlin
val student = Student(101, "ZhangSan", 30)
val (id, name, age) = student // 自动赋值给 id, name, age
println("id=$id, name=$name, age=$age")
```

### 相等性

`两个等号==和三个等号===`

两个等号 `==`：比较的是**对象的内容**是否相同，相当于 Java 的 equals()。`==` 的否定形式为 `!=` 。

三个等号 `===`：比较的是**对象的地址**是否相同（即判断是否为同一对象），相当于 Java 的 ==。`===` 的否定形式为 `!==` 。

```Kotlin
val student1 = Student(101, "ZhangSan", 30)
val student2 = Student(101, "ZhangSan", 30)
if (student1 == student2) { // true
    // ...
}
if (student1 === student2) { // false
    // ...
}
```

### operator

通过 operator 修饰「特定函数名」的函数，例如 plus 、get, 可以达到重载运算符的效果。

| 表达式   | 翻译为        |
| ----- | ---------- |
| a + b | a.plus(b)  |
| a - b | a.minus(b) |
| a * b | a.times(b) |
| a / b | a.div(b)   |

### 委托

委托，也就是委托模式，它是 23 种经典设计模式中的一种，又名`代理模式`，在委托模式中，有 2 个对象参与同一个请求的处理，接受请求的对象将请求委托给另一个对象来处理。委托模式中，有三个角色：约束、委托对象和被委托对象。Kotlin 直接支持委托模式，更加优雅，简洁。Kotlin 通过关键字 `by` 实现委托。

- **约束**：约束是接口或者抽象类，它定义了通用的业务类型，也就是需要被代理的业务。
- **被委托对象**：具体的业务逻辑执行者。
- **委托对象**：负责对真正角色的应用，将约束类定义的业务委托给具体的委托对象。

#### 类委托

类的委托即一个类中定义的方法实际是调用另一个类的对象的方法来实现的。

```Kotlin
// 约束类，我们约定的业务
interface IGamePlayer {
    fun play() // 打游戏
}

// 被委托对象，实现了我们约定的业务
class RealGamePlayer(private val name: String) : IGamePlayer {
    override fun play() {
        println("${name}开始打游戏")
    }
}

// 委托对象，通过关键字 by 建立委托类
class DelegateGamePlayer(private val player: IGamePlayer) : IGamePlayer by player

fun main(args: Array<String>) {
    val realGamePlayer = RealGamePlayer("张三")
    val delegateGamePlayer = DelegateGamePlayer(realGamePlayer)
    delegateGamePlayer.play() // 输出：张三开始打游戏
}
```

在 DelegateGamePlayer 声明中，by 子句表示，将 player 保存在 DelegateGamePlayer 的对象实例内部，而且编译器将会生成继承自 IGamePlayer 接口的所有方法，并将调用转发给 player。

>  可以通过类委托的模式来减少继承。

#### 属性委托

属性委托指的是一个类的某个属性值不是在类中直接进行定义，而是将其托付给一个代理类，从而实现对该类的属性统一管理。

属性委托语法格式：

```Kotlin
val/var <属性名>: <类型> by <表达式>
```

- var/val：属性类型(可变/只读)。
- 属性名：属性名称。
- 类型：属性的数据类型。
- 表达式：委托代理类。

by 关键字之后的表达式就是委托，属性的 get() 方法（以及 set() 方法）将被委托给这个对象的 getValue() 和 setValue() 方法。属性委托不必实现任何接口，但必须提供 getValue() 函数（对于 var 属性，还需要 setValue() 函数）。

```Kotlin
/**
 * 被委托类（委托代理类）
 * 该类需要包含 getValue() 方法和 setValue() 方法，且参数 thisRef 为进行委托的类的对象，property 为进行委托的属性的对象。
 */
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, 这里委托了 ${property.name} 属性"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$thisRef 的 ${property.name} 属性赋值为 $value")
    }
}

fun main(args: Array<String>) {
    var token: String by Delegate()
    println(token) // 访问该属性，调用 getValue() 函数

    token = "xxx"   // 调用 setValue() 函数
    println(token)
}
```

其中的参数解释如下：

- `thisRef` —— 必须与属性所有者类型（对于扩展属性——指被扩展的类型）相同或者是它的超类型。
- `property` —— 必须是类型 `KProperty<*>` 或其超类型。
- `value` —— 必须与属性同类型或者是它的子类型。

#### 属性委托的另一种实现方式

Kotlin 标准库中声明了 2 个含所需 `operator` 方法的 `ReadOnlyProperty / ReadWriteProperty` 接口。

```Kotlin
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```

`被委托类` 实现这两个接口其中之一就可以了，`val` 属性实现 `ReadOnlyProperty`, `var` 属性实现 `ReadWriteProperty`。 

```Kotlin
class Delegate2 : ReadWriteProperty<Any?, String> {
    override fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return ""
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
    }
}

var token2: String by Delegate2()
```

#### 标准库中提供的几个委托

- 延迟属性（lazy）: 其值只在首次访问时计算。
- 可观察属性（observable）: 监听器会收到有关此属性变更的通知。
- 把多个属性储存在一个映射（map）中，而不是每个存在单独的字段中。

**1、延迟属性 lazy**

lazy() 是一个函数，接受一个 Lambda 表达式作为参数，返回一个 Lazy <T> 实例的函数，返回的实例可以作为实现延迟属性的委托：第一次调用 get() 会执行已传递给 lazy() 的 lamda 表达式并记录结果，后续调用 get() 只是返回记录的结果。

```Kotlin
val lazyValue: String by lazy {
    println("computed!") // 第一次调用输出，第二次调用不执行
    "Hello"
}

fun main(args: Array<String>) {
    println(lazyValue) // 第一次执行，执行两次输出表达式
    println(lazyValue) // 第二次执行，只输出返回值
}

// 执行输出结果
computed!
Hello
Hello
```

**2、可观察属性 Observable**

observable 可以用于实现观察者模式。

Delegates.observable() 函数接受两个参数：第一个是初始化值，第二个是属性值被修改时的回调处理器 onChange。回调处理器有三个参数：被赋值的属性、旧值和新值：

```Kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("A") { property, oldValue, newValue ->
        println("旧值：$oldValue -> 新值：$newValue")
    }
}

fun main(args: Array<String>) {
    val user = User()
    user.name = "B"
    user.name = "C"
}

// 执行输出结果
旧值：A -> 新值：B
旧值：B -> 新值：C
```

**vetoable 函数**：`vetoable` 与 `observable` 一样，可以观察属性值的变化，不同的是，`vetoable` 可以通过`处理器函数来决定属性值是否生效`。

```Kotlin
// 声明一个 Int 类型的属性 vetoableProp, 如果新的值比旧值大，则生效，否则不生效。
var vetoableProp: Int by Delegates.vetoable(0){ _, oldValue, newValue ->
    // 如果新的值大于旧值，则生效
    newValue > oldValue
}
```

**3、把属性储存在映射中**

一个常见的用例是在一个映射（map）里存储属性的值。 这经常出现在像解析 JSON 或者做其他“动态”事情的应用中。 在这种情况下，你可以使用映射实例自身作为委托来实现委托属性。

```Kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int  by map
}

fun main(args: Array<String>) {
    // 构造函数接受一个映射参数
    val user = User(mapOf(
        "name" to "张三",
        "age"  to 18
    ))

    // 读取映射值
    println(user.name)
    println(user.age)
}
```

### 集合中函数式 API 操作符

#### 筛选过滤类

##### slice 系列

操作符可以取集合中一部分元素或者某个元素，最后组合成一个新元素。

- **`slice(indices: IntRange)`**：指定切片的起始位置和终止位置，将范围内的元素切出加入到新集合。

  ```kotlin
  val list = listOf(1, 2, 3, 4, 5).slice(IntRange(2, 4))
  println(list)
  
  // 打印结果：
  [3, 4, 5]
  ```

- **`slice(indices: Iterable<Int>)`**：指定下标分别切出对应的元素，放入新集合中。

  ```kotlin
  val list = listOf(1, 2, 3, 4, 5).slice(IntRange(2, 4))
  println(list)
  
  val list2 = listOf(1, 2, 3, 4, 5).slice(listOf(1, 3))
  println(list2)
  
  // 打印结果：
  [3, 4, 5]
  [2, 4]
  ```

##### filter 系列

- **`filter(predicate: (T) -> Boolean)`**：从一个集合筛选出符合条件的元素，并以一个新集合返回。

  ```kotlin
  val list = listOf(1, 2, 3, 4, 5).filter { it % 2 == 0 }
  println(list)
  
  // 打印结果：
  [2, 4]
  ```

- **`filterTo(destination: C, predicate: (T) -> Boolean)`**：从多个集合筛选出符合条件的元素，并最终用一个集合进行收集从每个集合筛选出的元素。

  ```kotlin
  val numberList1 = listOf(3, 6, 9, 12, 15)
  val numberList2 = listOf(5, 10, 15, 20, 25)
  val numberList3 = listOf(10, 20, 30, 40, 50)
  
  // filterTo 的 destination 是一个可变集合类型，所以这里使用 mutableListOf 初始化
  val newNumberList = mutableListOf<Int>().apply {
      numberList1.filterTo(this) {
          it % 5 == 0
      }
      numberList2.filterTo(this) {
          it % 5 == 0
      }
      numberList3.filterTo(this) {
          it % 20 == 0
      }
  }
  println(newNumberList)
  
  // 打印结果：
  [15, 5, 10, 15, 20, 25, 20, 40]
  ```

- **`filterIndexed(predicate: (index: Int, T) -> Boolean)`**：需要集合元素 `index` 参与筛选条件。

  ```kotlin
  val result = listOf(3, 6, 9, 12, 15).filterIndexed { index, item ->
      index > 2 && item % 2 == 0
  }
  println(result)
  
  // 打印结果：
  [12]
  ```

- **`filterIndexedTo(destination: C, predicate: (index: Int, T) -> Boolean)`**：从多个集合筛选出符合条件的元素，筛选条件需要 `index`，并最终用一个集合进行收集从每个集合筛选出的元素。

  ```kotlin
  val result = mutableListOf<Int>()
  listOf(0, 1, 2, 12, 4).filterIndexedTo(result) { index, item ->
      index == item
  }
  println(result)
  
  // 打印结果：
  [0, 1, 2, 4]
  ```

- **`filterIsInstance()`**：一个抽象类集合中含有多种子类型的元素，可以很方便筛选对应子类型的元素，并组成一个集合返回。

  ```kotlin
  val list = mutableListOf(1, "2", 3, "4", 5f, 6.0, "7")
  val result = list.filterIsInstance<Int>()
  println(result)
  
  // 打印结果：
  [1, 3]
  ```

- **`filterIsInstanceTo(destination: C)`**：适用于筛选多个集合的情况。

  ```kotlin
  val list = mutableListOf(1, "2", 3, "4", 5f, 6.0, "7")
  val result = mutableListOf<String>()
  list.filterIsInstanceTo(result)
  println(result)
  
  // 打印结果：
  [2, 4, 7]
  ```

- **`filterNot(predicate: (T) -> Boolean)`**、**`filterNotTo(destination: C, predicate: (T) -> Boolean)`**：从一个集合筛选出符合条件之外的元素，并以一个新集合返回。它是 filter 操作符取反操作。

- **`filterNotNull()`**、**`filterNotNullTo(destination: C)`**：filterNotNull 操作符可以过滤集合中为 null 的元素。同理 filterNotNullTo 才是真正过滤操作，但是需要从外部传入一个可变集合。

##### drop 系列

- **`drop(n: Int)`**：把集合元素去除一部分，drop 是顺序地删除，n 则表示顺序删除几个元素，最后返回剩余元素集合。

  ```kotlin
  println(listOf(1, 2, 3, 4, 5).drop(3))
  
  // 打印结果：
  [4, 5]
  ```

- **`dropLast(n: Int)`**：根据传入数值 n，表示从右到左倒序地删除 n 个集合中的元素，并返回集合中剩余的元素。

  ```kotlin
  println(listOf(1, 2, 3, 4, 5).dropLast(3))
  
  // 打印结果：
  [1, 2]
  ```

- **`dropWhile(predicate: (T) -> Boolean)`**：从集合的第一项开始去掉满足条件元素，这样操作一直持续到出现第一个不满足条件元素出现为止，返回剩余元素（可能剩余元素有满足条件的元素）。

  ```kotlin
  println(listOf(1, 2, 32, 40, 5).dropWhile { it < 20 })
  
  // 打印结果：
  [32, 40, 5]
  ```

- **`dropLastWhile(predicate: (T) -> Boolean)`**：从集合的最后一项开始去掉满足条件元素，这样操作一直持续到出现第一个不满足条件元素出现为止，返回剩余元素（可能剩余元素有满足条件的元素）。

  ```kotlin
   println(listOf(1, 2, 32, 40, 5).dropLastWhile { it < 20 })
  
  // 打印结果：
  [1, 2, 32, 40]
  ```


##### take 系列

- **`take(n: Int)`**：从原集合的第一项开始顺序取集合的元素，取 n 个元素，最后返回取出这些元素的集合。换句话说就是取集合前 n 个元素组成新的集合返回。

  ```kotlin
  println(listOf(1, 2, 32, 40, 5).take(3))
  
  // 打印结果：
  [1, 2, 32]
  ```

- **`takeLast(n: Int)`**：从原集合的最后一项开始倒序取集合的元素，取 n 个元素，最后返回取出这些元素的集合。

  ```kotlin
  println(listOf(1, 2, 32, 40, 5).takeLast(3))
  
  // 打印结果：
  [32, 40, 5]
  ```

- **`takeLastWhile(predicate: (T) -> Boolean)`**：从集合的最后一项开始取出满足条件元素，这样操作一直持续到出现第一个不满足条件元素出现为止，暂停取元素，返回取出元素的集合。

  ```kotlin
  println(listOf(1, 2, 32, 40, 5).takeLastWhile { it < 20 })
  
  // 打印结果：
  [5]
  ```

- **`takeWhile(predicate: (T) -> Boolean)`**：从集合的第一项开始取出满足条件元素，这样操作一直持续到出现第一个不满足条件元素出现为止，暂停取元素，返回取出元素的集合。

  ```kotlin
  println(listOf(1, 2, 32, 40, 5).takeWhile { it < 20 })
  
  // 打印结果：
  [1, 2]
  ```

##### distinct 系列

- `distinct`：去除集合中的重复元素。

  ```kotlin
  println(listOf(1, 2, 2, 1, 3).distinct())
  
  // 打印结果：
  [1, 2, 3]
  ```

- **`distinctBy(selector: (T) -> K)`**：根据操作元素后的结果去除重复元素。

  ```kotlin
  println(listOf(1, 2, 4, 3, 6).distinctBy { it % 2 })
  
  // 打印结果：
  [1, 2]
  ```

#### 并集类操作符

##### any、all、count、none 系列

- **`any()`**：判断是不是一个集合，若是，则再判断集合是否为空。若为空则返回 false，反之返回 true。若不是集合，则返回 hasNext。

  ```kotlin
  println(listOf(1, 2).any())
  
  // 打印结果：
  true
  ```

- **`any(predicate: (T) -> Boolean)`**：判断集合中是否存在满足条件的元素。若存在则返回 true，反之返回 false。

  ```kotlin
  println(listOf(1, 2, 32, 40, 5).any { it > 30 })
  
  // 打印结果：
  true
  ```

- **`all(predicate: (T) -> Boolean)`**：判断集合中的所有元素是否都满足条件。若是则返回 true，反之则返回 false。

  ```kotlin
  println(listOf(0, 2, 32, 40, 50).all { it % 2 == 0 })
  
  // 打印结果：
  true
  ```

- **`count()`**、**`count(predicate: (T) -> Boolean)`**：返回集合中的元素个数或查询集合中满足条件的元素个数。

  ```kotlin
  println(listOf(0, 2, 32, 40, 50).count())
  println(listOf(0, 2, 32, 40, 50).count { it > 10 })
  
  // 打印结果：
  5
  3
  ```

- **`none()`**、**`none(predicate: (T) -> Boolean)`**：如果一个集合是空集合，返回 true 或者集合中没有满足条件的元素，则返回 true。

  ```kotlin
  println(listOf(0, 2, 32, 40, 50).none()) 
  println(listOf(0, 2, 32, 40, 50).none { it > 50 })
  
  // 打印结果：
  false
  true
  ```

##### fold 系列

- **`fold(initial: R, operation: (acc: R, T) -> R)`**：在一个初始值的基础上，从第一项到最后一项通过一个函数累计所有的元素。

  ```kotlin
  println(listOf(0, 2, 4, 6).fold(10) { result, element ->
      result + element
  })
  
  // 打印结果：
  22
  ```

- **`foldIndexed(initial: R, operation: (index: Int, acc: R, T) -> R)`**：在一个初始值的基础上，从第一项到最后一项通过一个函数累计所有的元素，该函数的参数可以包含元素索引。

  ```kotlin
  println(listOf("h", "e", "l", "l", "o").foldIndexed("Say") { index, result, element ->
      "$result $element$index"
  })
  
  // 打印结果：
  Say h0 e1 l2 l3 o4
  ```

- **`foldRight(initial: R, operation: (T, acc: R) -> R)`**：在一个初始值的基础上，从最后项到第一项通过一个函数累计所有的元素，与 fold 类似，不过是从最后一项开始累计。

  ```kotlin
  println(listOf("1", "2", "3").foldRight("Say") { element, result ->
      "$result $element"
  })
  
  // 打印结果：
  Say 3 2 1
  ```

- **`foldRightIndexed(initial: R, operation: (index: Int, T, acc: R) -> R)`**：在一个初始值的基础上，从最后一项到第一项通过一个函数累计所有的元素，该函数的参数可以包含元素索引。

  ```kotlin
  println(listOf("1", "2", "3").foldRightIndexed("Say") { index, element, result ->
      "$result $element-$index"
  })
  
  // 打印结果：
  Say 3-2 2-1 1-0
  ```

##### forEach 系列

- **`forEach(action: (T) -> Unit)`**：集合元素的遍历操作符。

  ```kotlin
  listOf("1", "2", "3").forEach {
      print(it)
  }
  
  // 打印结果：
  123
  ```

- **`forEachIndexed(action: (index: Int, T) -> Unit)`**：集合中带元素下标的遍历操作。

  ```kotlin
  listOf("1", "2", "3").forEachIndexed { index, s ->
      print("$index-$s ")
  }
  
  // 打印结果：
  0-1 1-2 2-3
  ```

##### max、min 系列

- **`maxOrNull()`**：获取集合中最大的元素，若为空元素集合，则返回 null。

  ```kotlin
  println(listOf("B", "C", "A").maxOrNull())
  
  // 打印结果：
  C
  ```

- **`maxByOrNull(selector: (T) -> R)`**：根据给定的函数返回最大的一项，如果没有则返回 null。

  ```kotlin
  println(listOf(-2, 1, 0).maxByOrNull {
      it * -3
  })
  
  // 打印结果：
  -2
  ```

- **`maxWithOrNull(comparator: Comparator<in T>)`**：接受一个 Comparator 对象并且根据此 Comparator 对象返回最大元素。

  ```kotlin
  println(listOf("-20", "-102", "0").maxWithOrNull(compareBy {
      it.length
  }))
  
  // 打印结果：
  -102
  ```


> min 操作符作用与 max 相反，包括 minBy 和 minWith。

##### reduce 系列

- **`reduce(operation: (acc: S, T) -> S)`**：从集合中的第一项到最后一项的累计操作，与 fold 操作符的区别是没有初始值。

  ```kotlin
  println(listOf("Nice", "to", "meet", "you").reduce { result, element ->
      "$result $element"
  })
  
  // 打印结果：
  Nice to meet you
  ```

  > 其中 reduceIndexed，reduceRight，reduceRightIndexed 操作符都与前述的 fold 相关操作符类似，只是没有初始值。

- **`reduceOrNull(operation: (acc: S, T) -> S)`**：从集合中的第一项到最后一项的累计操作，如果集合为空，返回 null。

  > reduceRightOrNull 操作符作用等价于 reduceRight，不同的是当集合为空，返回 null。

##### sum 系列

- **`sum()`**：计算集合中所有元素累加的结果。

  ```kotlin
  println(listOf(1, 2, 3).sum())
  
  // 打印结果：
  6
  ```

- **`sumOf(selector: (T) -> Int)`**：计算集合所有元素通过某个函数转换后数据之和。

  ```kotlin
  println(listOf(1, 2, 3).sumOf { it * 2 })
  
  // 打印结果：
  12
  ```

#### 映射类操作符

##### flatMap 系列

- **`flatMap(transform: (T) -> Iterable<R>)`**：根据条件合并两个集合，组成一个新的集合。

  ```kotlin
  val strList = listOf("A", "B", "C")
  println(strList.flatMap { listOf(it.plus("1")) })
  
  // 打印结果：
  [A1, B1, C1]
  ```

- **`flatMapTo(destination: C, transform: (T) -> Iterable<R>)`**：多个集合的条件合并。

  ```kotlin
  val strList = listOf("A", "B", "C")
  val strList2 = listOf("a", "b", "c")
  val resultList = mutableListOf<String>().apply {
      strList.flatMapTo(this) {
          listOf(it.plus("1"))
      }
      strList2.flatMapTo(this) {
          listOf(it.plus("2"))
      }
  }
  println(resultList)
  
  // 打印结果：
  [A1, B1, C1, a2, b2, c2]
  ```

##### group 系列

- **`groupBy(keySelector: (T) -> K)`**：分组操作符，根据条件将集合拆分为一个 Map 类型集合。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.groupBy { if (it.startsWith("Java")) "MyJava" else "MyKotlin" })
  
  // 打印结果：
  {MyJava=[Java, JavaScript], MyKotlin=[Kotlin, C, C++]}
  ```

- **`groupByTo(destination: M, keySelector: (T) -> K)`**：分组操作，适用于多个集合的分组操作。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  val strList2 = listOf("15", "223", "45", "520", "18")
  val mutableMap = mutableMapOf<String, MutableList<String>>().apply {
      strList.groupByTo(this) {
          if (it.startsWith("Java")) "Java" else "Not Java"
      }
      strList2.groupByTo(this) {
          if (it.contains("2")) "2" else "Not 2"
      }
  }
  println(mutableMap)
  
  // 打印结果：
  {Java=[Java, JavaScript], Not Java=[Kotlin, C, C++], Not 2=[15, 45, 18], 2=[223, 520]}
  ```

- **`groupingBy(crossinline keySelector: (T) -> K)`**：对元素进行分组，然后一次将操作应用于所有分组。适用于对集合复杂分组的情况。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.groupingBy { it.startsWith("Java") }.eachCount())
  
  // 打印结果：
  {true=2, false=3}
  ```

  `Grouping` 支持以下操作：

  - eachCount() 计算每个组中的元素。
  - fold() 与 reduce() 对每个组分别执行 fold 与 reduce 操作，作为一个单独的集合并返回结果。
  - aggregate() 随后将给定操作应用于每个组中的所有元素并返回结果。 这是对 `Grouping` 执行任何操作的通用方法。当折叠或缩小不够时，可使用它来实现自定义操作。

##### map 系列

- **`map(transform: (T) -> R)`**：集合变换，遍历每个元素并执行给定表达式，最终形成新的集合。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.map { it.plus("-1") })
  
  // 打印结果：
  [Java-1, Kotlin-1, C-1, C++-1, JavaScript-1]
  ```

- **`mapTo(destination: C, transform: (T) -> R)`**：多个集合的元素转换。

  ```kotlin
  val strList = listOf("A", "B", "C")
  val strList2 = listOf("a", "b", "c")
  println(mutableListOf<String>().apply {
      strList.mapTo(this) { it.plus("-1") }
      strList2.mapTo(this) { it.plus("-2") }
  })
  
  // 打印结果：
  [A-1, B-1, C-1, a-2, b-2, c-2]
  ```

- **`mapIndexed(transform: (index: Int, T) -> R)`**：带有元素下标的集合转换。

  ```kotlin
  val strList = listOf("A", "B", "C")
  println(strList.mapIndexed { index, s -> "$index-$s" })
  
  // 打印结果：
  [0-A, 1-B, 2-C]
  ```

- **`mapIndexedTo(destination: C, transform: (index: Int, T) -> R)`**：带有元素下标的多个集合转换。

  ```kotlin
  val strList = listOf("A", "B", "C")
  val strList2 = listOf("a", "b", "c")
  println(mutableListOf<String>().apply {
      strList.mapIndexedTo(this) { index, s -> "$index-$s" }
      strList2.mapIndexedTo(this) { index, s -> "$index-$s" }
  })
  
  // 打印结果：
  [0-A, 1-B, 2-C, 0-a, 1-b, 2-c]
  ```

- **`mapNotNull(transform: (T) -> R?)`**：同 `map` 函数的作用相同，不过其过滤了集合转换后为 null 的元素。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.mapNotNull { if (it.startsWith("Java")) null else it })
  
  // 打印结果：
  [Kotlin, C, C++]
  ```

- **`mapNotNullTo(destination: C, transform: (T) -> R?)`**：同 mapNotNull 函数的作用相同，它适用于多集合操作场景。

- **`mapIndexedNotNull` 和 `mapIndexedNotNullTo`**：同理。

#### 元素类操作符

##### element 系列

- **`elementAt(index: Int)`**：获取集合指定下标的元素。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.elementAt(1)) // 打印：Kotlin
  ```

- **`elementAtOrElse(index: Int, defaultValue: (Int) -> T)`**：获取对应下标的集合元素。若下标越界，返回默认值。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.elementAtOrElse(10) { "unknown" }) // 打印：unknown
  ```

- **`elementAtOrNull(index: Int)`**：获取对应下标的集合元素。若下标越界，返回 null。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.elementAtOrNull(10)) // 打印：null
  ```

##### first、last 系列

- **`first()`**：获取集合第一个元素，若集合为空集合，这会抛出 NoSuchElementException 异常。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.first()) // 打印：Java
  ```

- **`first(predicate: (T) -> Boolean)`**：获取集合中指定条件的第一个元素。若不满足条件，抛异常。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.first { it.length > 5 }) // 打印：Kotlin
  ```

- **`firstOrNull()`**：获取集合第一个元素，若集合为空集合，返回 null。

- **`firstOrNull{}`**：获取集合满足条件的首个元素，若无则返回 null。

> 与 `first` 等操作符对应的是 `last` 等相关操作符，即取集合最后一个元素等。

##### find 系列

- **`find(predicate: (T) -> Boolean)`**：获取集合满足条件的首个元素，若无则返回 null，其实就是 firstOrNull{}。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.find { it.length > 5 }) // 打印：Kotlin
  ```

- **`findLast(predicate: (T) -> Boolean)`**：获取集合满足条件的最后一个元素，若无则返回 null，其实就是 lastOrNull{}。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.findLast { it.length > 5 }) // 打印：JavaScript
  ```

##### single 系列

- **`single()`**：当集合中只有一个元素时，返回该元素，否则抛异常。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.single()) // 打印：java.lang.IllegalArgumentException: List has more than one element.
  ```

- **`single(predicate: (T) -> Boolean)`**：找到集合中满足条件的元素，若只有单个元素满足条件，则返回该元素，否则抛异常。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.single { it.startsWith("K") }) // 打印：Kotlin
  ```

> **`singleOrNull` 或 `singleOrNull{}` 操作符** 只是将前述的抛异常改为返回 null。

##### component 系列

- **`component1()` ... `component5()`**：用于获取第 1 到第 5 个元素。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.component3()) // 打印：C
  ```

##### indexOf 系列

- **`indexOf(element: T)`**：返回指定元素的下标，若不存在，则返回 -1。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.indexOf("C")) // 打印：2
  ```

- **`indexOfFirst(predicate: (T) -> Boolean)`**：返回满足条件的第一个元素的下标，若不存在，则返回 -1。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.indexOfFirst { it.length > 5 }) // 打印：1
  ```

- **`indexOfLast(predicate: (T) -> Boolean)`**：返回满足条件的最后一个元素的下标，若不存在，则返回 -1。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.indexOfLast { it.length > 5 }) // 打印：4
  ```

#### 排序类操作符

##### reverse 系列

- **`reversed()`**：反转集合元素。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.reversed()) // 打印：[JavaScript, C++, C, Kotlin, Java]
  ```

##### sort 系列

- **`sorted()`**：对集合中的元素自然升序排序。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.sorted()) // 打印：[C, C++, Java, JavaScript, Kotlin]
  ```

- **`sortedDescending()`**：与 sorted 操作符相反，为倒序。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.sortedDescending()) // 打印：[Kotlin, JavaScript, Java, C++, C]
  ```

- **`sortedBy(crossinline selector: (T) -> R?)`**：根据条件升序，即把不满足条件的放在前面，满足条件的放在后面。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.sortedBy { it.length }) // 打印：[C, C++, Java, Kotlin, JavaScript]
  ```

- **`sortedByDescending(crossinline selector: (T) -> R?)`**：与 sortedBy 操作符相反，为倒序。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.sortedByDescending { it.length }) // 打印：[JavaScript, Kotlin, Java, C++, C]
  ```

#### 生成类操作符

##### partition 系列

- **`partition(predicate: (T) -> Boolean)`**：将一个集合按条件拆分为两个 pair 组成的新集合。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.partition { it.startsWith("Java") }) // 打印：([Java, JavaScript], [Kotlin, C, C++])
  ```

##### plus 系列

- **`plus(element: T)`**：合并两个集合中的元素，组成一个新的集合。也可以使用符号 `+`。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.plus(listOf(1, 2, 3))) // 打印：[Java, Kotlin, C, C++, JavaScript, 1, 2, 3]
  ```

- **`plusElement(element: T)`**：往集合中添加一个元素。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.plusElement("CSS")) // 打印：[Java, Kotlin, C, C++, JavaScript, CSS]
  ```

##### zip 系列

- **`zip(other: Array<out R>)`**：由两个集合按照相同的下标组成一个新集合。该新集合的类型是：`List<Pair>`。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.zip(listOf(1, 2, 3))) // 打印：[(Java, 1), (Kotlin, 2), (C, 3)]
  ```

- **`zipWithNext(transform: (a: T, b: T) -> R)`**：集合中相邻元素组成 `pairs`。

  ```kotlin
  val strList = listOf("Java", "Kotlin", "C", "C++", "JavaScript")
  println(strList.zipWithNext()) // 打印：[(Java, Kotlin), (Kotlin, C), (C, C++), (C++, JavaScript)]
  ```

  