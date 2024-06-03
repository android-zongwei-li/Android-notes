> Review：2024/6/1

# 一、特点

1、高效简洁，同样功能可以使用更少代码，少写样板代码。

2、更安全，可避免null指针异常错误。

3、可互操作，可以和Java代码互相调用。

4、结构化并发，协程让异步代码更易使用，简化后台任务管理。

# 二、涉及内容

![image-20211029223130848](images/image-20211029223130848.png)

# 一、Hello World

按照国际惯例，学习一门新的语言通常都是要从打印 Hello World 开始的

```kotlin
package main

fun main() {
    val msg: String = "Hello World"
    println(msg)
}
```

从这个简单的函数就可以列出 kotlin 和 Java 的几个不同点

1. 函数可以定义在文件的最外层，不需要把它放在类中
2. 用关键字 **fun** 来声明一个函数
3. 可以省略 `main` 方法的参数
4. 参数类型写在变量名之后，这有助于在类型自动推导时省略类型声明
5. 使用 `println` 代替了 `System.out.println` ，这是 kotlin 标准库提供的对 Java 标准库函数的简易封装
6. 可以省略代码结尾的分号

# 二、Package

kotlin 文件都以 **package** 开头，同个文件中定义的所有声明（类、函数和属性）都会被放到这个包中。同个包中的声明可以直接使用，不同包的声明需要导入后使用

包的声明要放在文件顶部，import 语句紧随其后

```kotlin
package base

import java.text.SimpleDateFormat
import java.util.*
```

kotlin 不区分导入的是类还是函数，可以使用 `import` 关键字导入任何种类的声明。此外也可以在包名称后加上 **.*** 来导入包中定义的所有声明，从而让包中定义的类、顶层函数、和属性都可以直接引用

```kotlin
package learn.package2

val index = 10

fun Test(status: Boolean) = println(status)

class Point(val x: Int, val y: Int) {

    val isEquals1: Boolean
        get() {
            return x == y
        }

    val isEquals2
        get() = x == y

    var isEquals3 = false
        get() = x > y
        set(value) {
            field = !value
        }

}
```

```kotlin
package learn.package1

import learn.package2.Point
import learn.package2.Test
import learn.package2.index

fun main() {
    val point = Point(10, index)
    Test(true)
}
```

Java 语言规定类要放到和包结构**匹配**的文件夹目录结构中，且文件名必须和类名相同。而 kotlin 允许把多个类放到同一个文件中，文件名也可以任意选择。kotlin 也没有文件夹目录施加任何限制，包层级结构不需要遵循同样的目录层级结构 ,但 kotlin 官方还是建议根据包声明把源码文件放到相应的目录中

如果包名出现命名冲突，可以使用  **as  关键字**在本地重命名冲突项来消歧义

```kotlin
import learn.package1.Point
import learn.package2.Point as PointTemp //PointTemp 可以用来表示 learn.package2.Point 了
```

kotlin 中也有一个类似的概念可以用来重命名现有类型，叫做类型别名。类型别名用于为现有类型提供一个替代名称，如果类型名称比较长，就可以通过引入一个较短或者更为简略的名称来方便记忆

类型别名不会引入新的类型，依然对应其底层类型，所以在下述代码中输出的 class 类型是一致的

```kotlin
class Base {

    class InnerClass

}

typealias BaseInner = Base.InnerClass

fun main() {
    val baseInner1 = Base.InnerClass()
    val baseInner2 = BaseInner()
    println(baseInner1.javaClass) //class test2.Base$InnerClass
    println(baseInner2.javaClass) //class test2.Base$InnerClass
}
```

# 三、变量与数据类型

笔记：变量与数据类型

# 四、函数

函数以关键字 fun 开头，函数名称紧随其后，再之后是用括号包裹起来的参数列表，如果函数有返回值，则再加上返回值类型，用一个冒号与参数列表隔开

```kotlin
//fun 用于表示声明一个函数，getNameLastChar 是函数名
//空括号表示该函数无传入参数，Char 表示函数的返回值类型是字符
fun getNameLastChar(): Char {
    return name.get(name.length - 1)
}

//带有两个不同类型的参数，一个是 String 类型，一个是 Int 类型
//返回值为 Int 类型
fun test1(str: String, int: Int): Int {
    return str.length + int
}
```

表达式函数体的返回值类型可以省略，返回值类型可以自动推断，这种用单行表达式与等号定义的函数叫做**表达式函数体**。但对于一般情况下的有返回值的**代码块函数体**，**必须显式地**写出返回类型和 return 语句

```kotlin
//getNameLastChar 函数的返回值类型以及 return 关键字是可以省略的
//返回值类型可以由编译器根据上下文进行推导
//因此，函数可以简写为以下形式
fun getNameLastChar() = name.get(name.length - 1)
```

如果函数没有有意义的返回值，则可以声明为 Unit ，也可以省略 Unit

以下三种写法都是等价的

```kotlin
fun test(str: String, int: Int): Unit {
    println(str.length + int)
}

fun test(str: String, int: Int) {
    println(str.length + int)
}

fun test(str: String, int: Int) = println(str.length + int)
```

## 1、命名参数

为了增强代码的可读性，kotlin 允许使用命名参数，即在调用某函数的时候，可以将函数参数名一起标明，从而明确地表达该参数的含义与作用，但是在指定了一个参数的名称后，之后的所有参数都需要标明名称

```kotlin
fun main() {
    //错误，在指定了一个参数的名称后，之后的所有参数都需要标明名称
    //compute(index = 110, "leavesC")
    compute(index = 120, value = "leavesC")
    compute(130, value = "leavesC")
}

fun compute(index: Int, value: String) {

}
```

## 2、默认参数值

可以在声明函数的时候指定参数的默认值，从而避免创建重载的函数

```kotlin
fun main() {
    compute(24)
    compute(24, "leavesC")
}

fun compute(age: Int, name: String = "leavesC") {

}
```

对于以上这个例子，如果按照常规的调用语法时，必须按照函数声明定义的参数顺序来给定参数，可以省略的只有排在末尾的参数

```kotlin
fun main() {
    //错误，不能省略参数 name
    // compute(24)
    // compute(24,100)
    //可以省略参数 value
    compute("leavesC", 24)
}

fun compute(name: String = "leavesC", age: Int, value: Int = 100) {}
```

如果使用命名参数，可以省略任何有默认值的参数，而且也可以按照任意顺序传入需要的参数

```kotlin
fun main() {
    compute(age = 24)
    compute(age = 24, name = "leavesC")
    compute(age = 24, value = 90, name = "leavesC")
    compute(value = 90, age = 24, name = "leavesC")
}

fun compute(name: String = "leavesC", age: Int, value: Int = 100) {

}
```

## 3、可变参数

通过使用 vararg 关键字声明可变参数，可变参数可以把任意个数的参数打包到数组中传给函数

例如，以下的几种函数调用方式都是正确的

```kotlin
fun main() {
    compute()
    compute("leavesC")
    compute("leavesC", "leavesc")
    compute("leavesC", "leavesc", "叶")
}

fun compute(vararg name: String) {
    name.forEach { println(it) }
}
```

在 Java 中，可以直接将数组传递给可变参数，而 **kotlin 要求显式地解包数组**，以便每个数组元素在函数中能够作为单独的参数来调用，这个功能被称为展开运算符，使用方式就是在数组参数前加一个 *

```kotlin
fun main() {
    val names = arrayOf("leavesC", "leavesc", "叶")
    compute(* names)
}

fun compute(vararg name: String) {
    name.forEach { println(it) }
}
```

## 4、局部函数

在函数中嵌套函数，被嵌套的函数称为局部函数

```kotlin
fun main() {
    compute("leavesC", "country")
}

fun compute(name: String, country: String) {
    fun check(string: String) {
        if (string.isEmpty()) {
            throw IllegalArgumentException("参数错误")
        }
    }
    check(name)
    check(country)
}
```

# 五、表达式和条件循环

## 1、语句和表达式

这里需要先区分“语句”和“表达式”这两个概念。语句是可以单独执行，能够产生实际效果的代码，表现为赋值逻辑、打印操作、流程控制等形式，Java 中的流程控制（if，while，for）等都是语句。表达式可以是一个值、变量、常量、操作符、或它们之间的组合，表达式可以看做是包含返回值的语句

例如，以下的赋值操作、流程控制、打印输出都是语句，其是作为一个整体存在的，且不包含返回值

```kotlin
val a = 10
for (i in 0..a step 2) {
    println(i)
}
```

再看几个表达式的例子

```kotlin
1       //字面表达式，返回 1

++1     //也属于表达式，自增会返回 2

//与 Java 不同，kotlin 中的 if 是作为表达式存在的，其可以拥有返回值
fun getLength(str: String?): Int {
    return if (str.isNullOrBlank()) 0 else str.length
}
```

## 2、If 表达式

if 的分支可以是代码块，最后的表达式作为该块的返回值

```kotlin
val maxValue = if (20 > 10) {
    println("maxValue is 20")
    20
} else {
    println("maxValue is 10")
    10
}
println(maxValue) //20
```

以下代码可以显示地看出 if 的返回值，完全可以用来替代 Java 中的**三元运算符**，因此 kotlin 并没有**三元运算符**

```kotlin
val list = listOf(1, 4, 10, 4, 10, 30)
val value = if (list.size > 0) list.size else null
println(value)  //6
```

如果 if 表达式分支是用于执行某个命令，那么此时的返回值类型就是 Unit ，此时的 if 语句就看起来和 Java 的一样了

```kotlin
val value1 = if (list.size > 0) println("1") else println("2")
println(value1.javaClass)   //class kotlin.Unit
```

如果将 if 作为表达式而不是语句（例如：返回它的值或者把它赋给变量），该表达式需要有 else 分支

## 3、when 表达式

when 表达式与 Java 中的 **switch/case** 类似，但是要强大得多。when 既可以被当做表达式使用也可以被当做语句使用，when 将参数和所有的分支条件顺序比较直到某个分支满足条件，然后它会运行右边的表达式。如果 when 被当做表达式来使用，符合条件的分支的值就是整个表达式的值，如果当做语句使用， 则忽略个别分支的值。与 Java 的 switch/case 不同之处是 when 表达式的参数可以是任何类型，并且分支也可以是一个条件

和 if 一样，when 表达式每一个分支可以是一个代码块，它的值是代码块中最后的表达式的值，如果其它分支都不满足条件将会求值于 else 分支

如果 when 作为一个表达式使用，则必须有 else 分支， 除非编译器能够检测出所有的可能情况都已经覆盖了。如果很多分支需要用相同的方式处理，则可以把多个分支条件放在一起，用逗号分隔

```kotlin
val value = 2
when (value) {
    in 4..9 -> println("in 4..9") //区间判断
    3 -> println("value is 3")    //相等性判断
    2, 6 -> println("value is 2 or 6")    //多值相等性判断
    is Int -> println("is Int")   //类型判断
    else -> println("else")       //如果以上条件都不满足，则执行 else
}
```

```kotlin
fun main() {
    //返回 when 表达式
    fun parser(obj: Any): String =
            when (obj) {
                1 -> "value is 1"
                "4" -> "value is string 4"
                is Long -> "value type is long"
                else -> "unknown"
            }
    println(parser(1))
    println(parser(1L))
    println(parser("4"))
    println(parser(100L))
    println(parser(100.0))
}

value is 1
value type is long
value is string 4
value type is long
unknown
```

此外，when 语句也可以不带参数来使用

```kotlin
when {
    1 > 5 -> println("1 > 5")
    3 > 1 -> println("3 > 1")
}
```

## 4、for 循环

和 Java 中的 for 循环最为类似的形式是

```kotlin
val list = listOf(1, 4, 10, 34, 10)
for (value in list) {
    println(value)
}
```

通过索引来遍历

```kotlin
val items = listOf("H", "e", "l", "l", "o")
//通过索引来遍历List
for (index in items.indices) {
    println("${index}对应的值是：${items[index]}")
}
```

也可以在每次循环中获取当前索引和相应的值

```kotlin
val list = listOf(1, 4, 10, 34, 10)
for ((index, value) in list.withIndex()) {
    println("index : $index , value :$value")
}
```

也可以自定义循环区间

```kotlin
for (index in 2..10) {
    println(index)
}
```

## 5、while 和 do/while 循环

while 和 do/while 与 Java 中的区别不大

```kotlin
val list = listOf(1, 4, 15, 2, 4, 10, 0, 9)
var index = 0
while (index < list.size) {
    println(list[index])
    index++
}
```

```kotlin
val list = listOf(1, 4, 15, 2, 4, 10, 0, 9)
var index = 0
do {
    println(list[index])
    index++
} while (index < list.size)
```

## 6、返回和跳转

kotlin 有三种结构化跳转表达式：

- return 默认从最直接包围它的函数或者匿名函数返回
- break  终止最直接包围它的循环
- continue  继续下一次最直接包围它的循环

在 kotlin 中任何表达式都可以用标签（label）来标记，标签的格式为标识符后跟 @ 符号，例如：**abc@ 、fooBar@** 都是有效的标签

```kotlin
fun main() {
    fun1()
}

fun fun1() {
    val list = listOf(1, 4, 6, 8, 12, 23, 40)
    loop@ for (it in list) {
        if (it == 8) {
            continue
        }
        if (it == 23) {
            break@loop
        }
        println("value is $it")
    }
    println("function end")
}
```

```kotlin
value is 1
value is 4
value is 6
value is 12
function end
```

kotlin 有函数字面量、局部函数和对象表达式。因此 kotlin 的函数可以被嵌套

标签限制的 return 允许从外层函数返回，最重要的一个用途就是从 lambda 表达式中返回。通常情况下使用隐式标签更方便，该标签与接受该 lambda 的函数同名

```kotlin
fun main() {
    fun1()
    fun2()
    fun3()
    fun4()
    fun5()
}

fun fun1() {
    val list = listOf(1, 4, 6, 8, 12, 23, 40)
    list.forEach {
        if (it == 8) {
            return
        }
        println("value is $it")
    }
    println("function end")

//    value is 1
//    value is 4
//    value is 6
}

fun fun2() {
    val list = listOf(1, 4, 6, 8, 12, 23, 40)
    list.forEach {
        if (it == 8) {
            return@fun2
        }
        println("value is $it")
    }
    println("function end")

//    value is 1
//    value is 4
//    value is 6
}

//fun3() 和 fun4() 中使用的局部返回类似于在常规循环中使用 continue
fun fun3() {
    val list = listOf(1, 4, 6, 8, 12, 23, 40)
    list.forEach {
        if (it == 8) {
            return@forEach
        }
        println("value is $it")
    }
    println("function end")
    
//    value is 1
//    value is 4
//    value is 6
//    value is 12
//    value is 23
//    value is 40
//    function end
}

fun fun4() {
    val list = listOf(1, 4, 6, 8, 12, 23, 40)
    list.forEach loop@{
        if (it == 8) {
            return@loop
        }
        println("value is $it")
    }
    println("function end")

//    value is 1
//    value is 4
//    value is 6
//    value is 12
//    value is 23
//    value is 40
//    function end
}

fun fun5() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) {
            //局部返回到匿名函数的调用者，即 forEach 循环
            return
        }
        println("value is $value")
    })
    println("function end")
}
```

# 六、区间（范围使用）

表示范围，主要包括：in、downTo、step、until

Ranges 表达式使用一个 **..**  操作符来声明一个闭区间，它被用于定义实现一个 RangTo 方法

以下三种声明方式都是等价的

```kotlin
var index = 5

if (index >= 0 && index <= 10) {

}

if (index in 0..10) {

}

if (index in 0.rangeTo(10)) {

}
```

数字类型的 ranges 在被迭代时，编译器会将它们转换为与 Java 中使用 index 的 for 循环的相同字节码的方式来进行优化

Ranges 默认会自增长，所以像以下的代码就不会被执行

```kotlin
for (index in 10..0) {
    println(index)
}
```

可以改用 downTo 函数来将之改为递减

```kotlin
for (index in 10 downTo 0) {
    println(index)
}
```

可以在 ranges 中使用 step 来定义每次循环递增或递减的长度：

```kotlin
for (index in 1..8 step 2){
    println(index)
}
for (index in 8 downTo 1 step 2) {
    println(index)
}
```

以上声明的都是闭区间，如果想声明的是开区间，可以使用 until 函数：

```kotlin
for (index in 0 until 4){
    println(index)
}
```

扩展函数 `reversed()` 可用于返回将区间反转后的序列

```kotlin
val rangeTo = 1.rangeTo(3)
for (i in rangeTo) {
    println(i) //1  2  3
}
for (i in rangeTo.reversed()) {
    println(i) //3  2  1
}
```

# 七、修饰符

## 1、final 和  open

kotlin 中的类和方法默认都是 final 的，即不可继承的，如果想允许创建一个类的子类，需要使用 open 修饰符来标识这个类，此外，也需要为每一个希望被重写的属性和方法添加 open 修饰符

```kotlin
open class View {
    open fun click() {

    }
	//不能在子类中被重写
    fun longClick() {

    }
}

class Button : View() {
    override fun click() {
        super.click()
    }
}
```

如果重写了一个基类或者接口的成员，重写了的成员同样默认是 open 的。例如，如果 Button 类是 open 的，则其子类也可以重写其 click() 方法

```kotlin
open class Button : View() {
    override fun click() {
        super.click()
    }
}

class CompatButton : Button() {
    override fun click() {
        super.click()
    }
}
```

如果不想要类的子类重写该方法的实现，可以显式地将重写的成员标记为 final

```kotlin
open class Button : View() {
    override final fun click() {
        super.click()
    }
}
```

## 2、public

public 修饰符是限制级最低的修饰符，是默认的修饰符。如果一个定义为 public  的成员被包含在一个 private  修饰的类中，那么这个成员在这个类以外也是不可见的

## 3、protected

protected 修饰符只能被用在类或者接口中的成员上。在 Java 中，可以从同一个包中访问一个 protected 的成员，但对于 kotlin 来说，protected 成员只在该类和它的子类中可见。此外，protected 不适用于顶层声明

## 4、internal

一个定义为 internal 的包成员，对其所在的整个 module 可见，但对于其它 module 而言就是不可见的了。例如，假设我们想要发布一个开源库，库中包含某个类，我们希望这个类对于库本身是全局可见的，但对于外部使用者来说它不能被引用到，此时就可以选择将其声明为 internal 的来实现这个目的

> 根据 Jetbrains 的定义，一个 module  应该是一个单独的功能性的单位，可以看做是一起编译的 kotlin 文件的集合，它应该是可以被单独编译、运行、测试、debug 的。相当于在 Android Studio 中主工程引用的 module

## 5、private

private  修饰符是限制级最高的修饰符，kotlin 允许在顶层声明中使用 private 可见性，包括类、函数和属性，这表示只在自己所在的文件中可见，所以如果将一个类声明为 private，就不能在定义这个类之外的地方中使用它。此外，如果在一个类里面使用了 private  修饰符，那访问权限就被限制在这个类里面，继承这个类的子类也不能使用它。所以如果类、对象、接口等被定义为 private，那么它们只对被定义所在的文件可见。如果被定义在了类或者接口中，那它们只对这个类或者接口可见

## 6、总结

| 修饰符         | 类成员       | 顶层声明     |
| -------------- | ------------ | ------------ |
| public（默认） | 所有地方可见 | 所有地方可见 |
| internal       | 模块中可见   | 模块中可见   |
| protected      | 子类中可见   |              |
| private        | 类中可见     | 文件中可见   |

# 八、空安全

## 1、可空性

在 kotlin 中，类型系统将一个引用分为可以容纳  null （可空引用）或者不能容纳 null（非空引用）两种类型。 例如，String 类型的常规变量不能指向 null 

```kotlin
var name: String = "leavesC"
//编译错误
//name = null
```

如果希望一个变量可以储存 null 引用，需要显式地在类型名称后面加上问号

```kotlin
var name: String? = "leavesC"
name = null
```

问号可以加在任何类型的后面来表示这个类型的变量可以存储 null 引用：`Int?、Doubld? 、Long?` 等

kotlin 对可空类型的显式支持有助于防止 **NullPointerException** 导致的异常问题，编译器不允许不对可空变量做 null 检查就直接调用其属性。这个强制规定使得开发者必须在编码初期就考虑好变量的可赋值范围并为其各个情况做好分支处理

```kotlin
fun check(name: String?): Boolean {
    //error，编译器不允许不对 name 做 null 检查就直接调用其属性
    return name.isNotEmpty()
}
```

正确的做法是显式地进行 null 检查

```kotlin
fun check(name: String?): Boolean {
    if (name != null) {
        return name.isNotEmpty()
    }
    return false
}
```

>  // 注：安全调用符还可以链式调用
> a?.b?.c?.d
>  // 假设a不为null，才继续往下调用，以此类推
>  // 若该链式调用中任何一个属性为null，整个表达式都会返回null。

## 2、安全调用运算符：?.

安全调用运算符：`?.` 允许把一次 null 检查和一次方法调用合并为一个操作，如果变量值非空，则方法或属性会被调用，否则直接返回 null

例如，以下两种写法是完全等同的：

```kotlin
fun check(name: String?) {
    if (name != null) {
        println(name.toUpperCase())
    } else {
        println(null)
    }
}

fun check(name: String?) {
    println(name?.toUpperCase())
}
```

## 3、Elvis 运算符：?:

Elvis 运算符：`?:` 用于替代 `?.` 直接返回默认值 null 的情况，Elvis 运算符接收两个运算数，如果第一个运算数不为 null ，运算结果就是其运算结果值，如果第一个运算数为 null ，运算结果就是第二个运算数

例如，以下两种写法是完全等同的：

```kotlin
fun check(name: String?) {
    if (name != null) {
        println(name)
    } else {
        println("default")
    }
}

fun check(name: String?) {
    println(name ?: "default")
}
```

## 4、安全转换运算符：as?

安全转换运算符：`as?` 用于把值转换为指定的类型，如果值不适合该类型则返回 null

```kotlin
fun check(any: Any?) {
    val result = any as? String
    println(result ?: println("is not String"))
}
```

## 5、非空断言：!!

非空断言用于把任何值转换为非空类型，如果对 null 值做非空断言，则会抛出异常

```kotlin
fun main() {
    var name: String? = "leavesC"
    check(name) //7

    name = null
    check(name) //kotlinNullPointerException
}

fun check(name: String?) {
    println(name!!.length)
}
```

## 6、可空类型的扩展

为可空类型定义扩展函数是一种更强大的处理 null 值的方式，可以允许接收者为 null 的调用，并在该函数中处理 null ，而不是在确保变量不为 null 之后再调用它的方法

例如，如下方法可以被正常调用而不会发生空指针异常

```kotlin
val name: String? = null
println(name.isNullOrEmpty()) //true
```

`isNullOrEmpty()` 的方法签名如下所示，可以看到这是为可空类型 **CharSequence?** 定义的扩展函数，方法中已经处理了方法调用者为 null 的情况

```kotlin
@kotlin.internal.InlineOnly
public inline fun CharSequence?.isNullOrEmpty(): Boolean {
    contract {
        returns(false) implies (this@isNullOrEmpty != null)
    }
    return this == null || this.length == 0
}
```

## 7、平台类型

平台类型是 kotlin 对 java 所作的一种平衡性设计。kotlin 将对象的类型分为了可空类型和不可空类型两种，但 java 平台的一切对象类型均为可空的，当在 kotlin 中引用 java 变量时，如果将所有变量均归为可空类型，最终将多出许多 null 检查；如果均看成不可空类型，那么就很容易就写出忽略了NPE 风险的代码。为了平衡两者，kotlin 引入了平台类型，即当在 kotlin 中引用 java 变量值时，既可以将之看成可空类型，也可以将之看成不可空类型，由开发者自己来决定是否进行 null 检查

# 九、类型的检查与转换

## 1、类型检查

**is 与 !is** 操作符用于在运行时检查对象是否符合给定类型：

```kotlin
fun main() {
    val strValue = "leavesC"
    parserType(strValue) //value is String , length : 7
    val intValue = 100
    parserType(intValue) //value is Int , toLong : 100
    val doubleValue = 100.22
    parserType(doubleValue) //value !is Long
    val longValue = 200L
    parserType(longValue) //unknown
}

fun parserType(value: Any) {
    when (value) {
        is String -> println("value is String , length : ${value.length}")
        is Int -> println("value is Int , toLong : ${value.toLong()}")
        !is Long -> println("value !is Long")
        else -> println("unknown")
    }
}
```

同时，is 操作符也附带了一个智能转换功能。在许多情况下，不需要在 kotlin 中使用显式转换操作符，因为编译器跟踪不可变值的 is 检查以及显式转换，并在需要时自动插入安全的转换

例如，在上面的例子中，当判断 value 为 String 类型通过时，就可以直接将 value 当做 String 类型变量并调用其内部属性，这个过程就叫做智能转换

编译器会指定根据上下文环境，将变量智能转换为合适的类型

```kotlin
if (value !is String) return
//如果 value 非 String 类型时直接被 return 了，所以此处可以直接访问其 length 属性
println(value.length)

// || 右侧的 value 被自动隐式转换为字符串，所以可以直接访问其 length 属性
if (value !is String || value.length > 0) {

}

// && 右侧的 value 被自动隐式转换为字符串，所以可以直接访问其 length 属性
if (value is String && value.length > 0) {

}
```

## 2、不安全的转换操作符

如果转换是不可能的，转换操作符 `as` 会抛出一个异常。因此，我们称之为不安全的转换操作符

```kotlin
fun main() {
    parserType("leavesC") //value is String , length is 7
    parserType(10) //会抛出异常 ClassCastException
}

fun parserType(value: Any) {
    val tempValue = value as String
    println("value is String , length is ${tempValue.length}")
}
```

需要注意的是，null 不能转换为 String 变量，因为该类型**不是可空的**

因此如下转换会抛出异常

```kotlin
val x = null
val y: String = x as String //会抛出异常 TypeCastException
```

为了匹配安全，可以转换的类型声明为可空类型

```kotlin
val x = null
val y: String? = x as String?
```

## 3、安全的转换操作符

可以使用安全转换操作符 as? 来避免在转换时抛出异常，它在失败时返回 null

```kotlin
val x = null
val y: String? = x as? String
```

尽管以上例子 as? 的右边是一个非空类型的 String，但是其转换的结果是可空的

# 十、类

见笔记：类与接口.md

# 集合

## 1、只读集合与可变集合

kotlin 的集合设计和 Java 不同的另一项特性是：kotlin 把访问数据的接口和修改集合数据的接口分开了，`kotlin.collections.Collection` 接口提供了**遍历集合元素、获取集合大小、判断集合是否包含某元素**等操作，但这个接口没有提供**添加和移除元素**的方法。`kotlin.collections.MutableCollection` 接口继承于 `kotlin.collections.Collection` 接口，扩展出了用于**添加、移除、清空元素**的方法

就像 kotlin 对 `val` 和 `var` 的区分一样，只读集合接口与可变集合接口的分离能提高对代码的可控性，如果函数接收 `Collection` 作为形参，那么就可以知道该函数不会修改集合，而只是对数据进行读取

以下是用来创建不同类型集合的函数

| 集合元素 | 只读   | 可变                                              |
| -------- | ------ | ------------------------------------------------- |
| List     | listOf | mutableListOf、arrayListOf                        |
| Set      | setOf  | mutableSetOf、hashSetOf、linkedSetOf、sortedSetOf |
| Map      | mapOf  | mutableMapOf、hashMapOf、linkedMapOf、sortedMapOf |

```kotlin
val list = listOf(10, 20, 30, 40)
//不包含 add 方法
//list.add(100)
println(list.size)
println(list.contains(20))

val mutableList = mutableListOf("leavesC", "leavesc", "叶")
mutableList.add("Ye")
println(mutableList.size)
println(mutableList.contains("leavesC"))
```

## 2、集合与 Java

因为 Java 并不会区分只读集合与可变集合，即使 kotlin 中把集合声明为只读的， Java 代码也可以修改这个集合，而 Java 代码中的集合对 kotlin 来说也是可变性未知的，kotlin 代码可以将之视为只读的或者可变的，包含的元素也是可以为 null 或者不为 null 的

例如，在 Java 代码中 names 这么一个 List<String> 类型的变量

```java
public class JavaMain {

    public static List<String> names = new ArrayList<>();

    static {
        names.add("leavesC");
        names.add("Ye");
    }

}
```

在 kotlin 中可以用以下四种方式来引用变量 names 

```kotlin
val list1: List<String?> = JavaMain.names

val list2: List<String> = JavaMain.names

val list3: MutableList<String> = JavaMain.names

val list4: MutableList<String?> = JavaMain.names
```

## 3、只读集合的可变性

只读集合不一定就是不可变的。例如，假设存在一个拥有只读类型接口的对象，该对象存在两个不同的引用，一个只读，一个可变，当可变引用修改了该对象后，这对只读引用来说就相当于“只读集合被修改了”，因此只读集合并不总是线程安全的。如果需要在多线程环境下处理数据，需要保证正确地同步了对数据的访问，或者使用支持并发访问的数据结构

例如，list1 和 list1 引用到同一个集合对象， list3 对集合的修改同时会影响到 list1

```kotlin
val list1: List<String> = JavaMain.names
val list3: MutableList<String> = JavaMain.names
list1.forEach { it -> println(it) } //leavesC Ye
list3.forEach { it -> println(it) } //leavesC Ye
for (index in list3.indices) {
    list3[index] = list3[index].toUpperCase()
}
list1.forEach { it -> println(it) } //LEAVESC YE
```

## 4、集合与可空性

集合的可空性可以分为三种：

1. 可以包含为 null 的集合元素
2. 集合本身可以为 null
3. 集合本身可以为 null，且可以包含为 null 的集合元素

例如，intList1 可以包含为 null 的集合元素，但集合本身不能指向 null；intList2 不可以包含为 null 的集合元素，但集合本身可以指向 null；intList3 可以包含为 null 的集合元素，且集合本身能指向 null

```kotlin
//List<Int?> 是能持有 Int? 类型值的列表
val intList1: List<Int?> = listOf(10, 20, 30, 40, null)
//List<Int>? 是可以为 null 的列表
var intList2: List<Int>? = listOf(10, 20, 30, 40)
intList2 = null
//List<Int?>? 是可以为 null 的列表，且能持有 Int? 类型值
var intList3: List<Int?>? = listOf(10, 20, 30, 40, null)
intList3 = null
```

# 十六、扩展函数和扩展属性

## 1、扩展函数

扩展函数用于为一个类增加一种新的行为。扩展函数的用途就类似于在 Java 中实现的静态工具方法。而在 kotlin 中使用扩展函数的一个优势是不需要在调用方法的时候把整个对象当作参数传入，扩展函数就像是属于这个类本身的一样，可以使用 this 关键字并直接调用其所有 public 方法

扩展函数不允许打破它的封装性，和在类内部定义的方法不同的是，**扩展函数不能访问私有的或是受保护的成员**

```kotlin
//为 String 类声明一个扩展函数 lastChar() ，用于返回字符串的最后一个字符
//get方法是 String 类的内部方法，length 是 String 类的内部成员变量，在此处可以直接调用
fun String.lastChar() = get(length - 1)

//为 Int 类声明一个扩展函数 doubleValue() ，用于返回其两倍值
//this 关键字代表了 Int 值本身
fun Int.doubleValue() = this * 2
```

之后，就可以像调用类本身内部声明的方法一样，直接调用扩展函数

```kotlin
fun main() {
    val name = "leavesC"
    println("$name lastChar is: " + name.lastChar())

    val age = 24
    println("$age doubleValue is: " + age.doubleValue())
}
```

如果需要声明一个静态的扩展函数，则必须将其定义在伴生对象上，这样就可以在没有 Namer 实例的情况下调用其扩展函数，就如同在调用 Java 的静态函数一样

```kotlin
class Namer {

    companion object {

        val defaultName = "mike"

    }

}

fun Namer.Companion.getName(): String {
    return defaultName
}

fun main() {
    Namer.getName()
}
```

需要注意的是，如果扩展函数声明于 class 内部，则该扩展函数只能该类和其子类内部调用，因为此时相当于声明了一个非静态函数，外部无法引用到。所以一般都是将扩展函数声明为全局函数

## 2、扩展属性

扩展函数也可以用于属性

```kotlin
// 扩展函数也可以用于属性
// 为 String 类新增一个属性值 customLen
var String.customLen: Int
    get() = length
    set(value) {
        println("set")
    }

fun main() {
    val name = "leavesC"
    println(name.customLen)
    name.customLen = 10
    println(name.customLen)
    //7
    //set
    //7
}
```

## 3、不可重写的扩展函数

看以下例子，子类 Button 重写了父类 View 的 click() 函数，此时如果声明一个 View 变量，并赋值为 Button 类型的对象，调用的 click() 函数将是 Button 类重写的方法

```kotlin
fun main() {
    val view: View = Button()
    view.click() //Button clicked
}

open class View {
    open fun click() = println("View clicked")
}

class Button : View() {
    override fun click() = println("Button clicked")
}
```

对于扩展函数来说，与以上的例子却不一样。如果基类和子类都分别定义了一个同名的扩展函数，此时要调用哪个扩展函数是由变量的静态类型来决定的，而非这个变量的运行时类型

```kotlin
fun main() {
    val view: View = Button()
    view.longClick() //View longClicked
}

open class View {
    open fun click() = println("View clicked")
}

class Button : View() {
    override fun click() = println("Button clicked")
}

fun View.longClick() = println("View longClicked")

fun Button.longClick() = println("Button longClicked")
```

此外，如果一个类的成员函数和扩展函数有相同的签名，**成员函数会被优先使用**

扩展函数并不是真正地修改了原来的类，其底层其实是以静态导入的方式来实现的。扩展函数可以被声明在任何一个文件中，因此有个通用的实践是把一系列有关的函数放在一个新建的文件里

需要注意的是，扩展函数不会自动地在整个项目范围内生效，如果需要使用到扩展函数，需要进行导入

## 4、可空接收者

可以为可空的接收者类型定义扩展，即使接受者为 null，使得开发者在调用扩展函数前不必进行判空操作，且可以通过 `this == null` 来检查接收者是否为空

```kotlin
fun main() {
    var name: String? = null
    name.check() //this == null
    name = "leavesC"
    name.check() //this != null
}

fun String?.check() {
    if (this == null) {
        println("this == null")
        return
    }
    println("this != null")
}
```

# 十七、Lambda 表达式

Lambda 表达式本质上就是可以传递给其它函数的一段代码，通过 Lambda 表达式可以把通用的代码结构抽取成库函数，也可以把 Lambda 表达式存储在一个变量中，把这个变量当做普通函数对待

```kotlin
// 由于存在类型推导，所以以下三种声明方式都是完全相同的
val plus1: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
val plus2: (Int, Int) -> Int = { x, y -> x + y }
val plus3 = { x: Int, y: Int -> x + y }
println(plus3(1, 2))
```

1. 一个 Lambda 表达式始终用花括号包围，通过箭头把实参列表和函数体分开
2. 如果 Lambda 声明了函数类型，那么就可以省略函数体的类型声明
3. 如果 Lambda 声明了参数类型，且返回值支持类型推导，那么就可以省略函数类型声明

虽然说倾向于尽量避免让 Lambda 表达式引用外部变量以避免副作用，但有些情况下让 Lambda 引用外部变量也可以简化计算结构。访问了外部环境变量的 Lambda 表达式称之为闭包，闭包可以被当做参数传递或者直接使用。与 Java 不同，kotlin 中的闭包不仅可以访问外部变量也可以对其进行修改

例如，假设需要一个计算总和的方法，每次调用函数时都返回当前的总和大小。方法外部不提供保存当前总和的变量，由 Lambda 表达式内部进行存储

```kotlin
fun main() {
    val sum = sumFunc()
    println(sum(10)) //10
    println(sum(20)) //30
    println(sum(30)) //60
}

fun sumFunc(): (Int) -> Int {
    var base = 0
    return fun(va: Int): Int {
        base += va
        return base
    }
}
```

此外，kotlin 也支持一种自动运行的语法

```kotlin
{ va1: Int, va2: Int -> println(va1 + va2) }(10, 20)
```

Lambda 表达式最常见的用途就是和集合一起工作，看以下例子

要从一个人员列表中取出年龄最大的一位

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("leavesC", 24), Person("Ye", 22))
    println(people.maxBy { it.age }) //Person(name=leavesC, age=24)
}
```

当中，库函数 maxBy 可以在任何集合上调用，其需要一个实参：一个函数，用于指定要用来进行比较的函数。花括号中的代码 `{ it.age }` 就是实现了这个逻辑的 Lambda 表达式

上述 maxBy 函数的实参是简化后的写法，这里来看下 maxBy 函数的简化过程

最原始的语法声明应该是这样的，用括号包裹着 Lambda 表达式

```kotlin
println(people.maxBy({ p: Person -> p.age }))
```

kotlin 有一种语法约定，如果 Lambda 表达式是函数调用的最后一个实参，可以将之放到括号的外边

```kotlin
 println(people.maxBy() { p: Person -> p.age })
```

当 Lamdba 表达式是函数唯一的实参时，可以去掉调用代码中的空括号对

```kotlin
 println(people.maxBy { p: Person -> p.age })
```

当 Lambda 表达式的参数类型是可以被推导出来时就可以省略声明参数类型

```kotlin
println(people.maxBy { p -> p.age })
```

如果当前上下文期待的是只有一个参数的 Lambda 表达式且参数类型可以被推断出来，就会为该参数生成一个默认名称：it

```kotlin
 println(people.maxBy { it.age })
```

kotlin 和 Java 的一个显著区别就是，在 kotlin 中函数内部的 Lambda 表达式不会仅限于访问函数的参数以及 final 变量，在 Lambda 内部也可以访问并修改非 final 变量

从 Lambda 内部访问外部变量，我们称这些变量被 Lambda 捕捉。当捕捉 final 变量时，变量值和使用这个值的 Lambda 代码一起存储，对非 final 变量来说，其值被封装在一个特殊的包装器中，对这个包装器的引用会和 Lambda 代码一起存储

```kotlin
var number = 0
val list = listOf(10, 20, 30, 40)
list.forEach {
    if (it > 20) {
        number++
    }
}
println(number) //2
```

## 成员引用

成员引用用于创建一个调用单个方法或者访问单个属性的函数值，通过双冒号把类名称和要引用的成员（一个方法或者一个属性）名称分隔开

成员引用的一个用途就是：如果要当做参数传递的代码块已经被定义成了函数，此时不必专门创建一个调用该函数的 Lambda 表达式，可以直接通过成员引用的方式来传递该函数（也可以传递属性）。此外，成员引用对扩展函数一样适用

```kotlin
data class Person(val name: String, val age: Int) {

    val myAge = age

    fun getPersonAge() = age
}

fun Person.filterAge() = age

fun main() {
    val people = listOf(Person("leavesC", 24), Person("Ye", 22))
    println(people.maxBy { it.age })    //Person(name=leavesC, age=24)
    println(people.maxBy(Person::age))  //Person(name=leavesC, age=24)
    println(people.maxBy(Person::myAge))  //Person(name=leavesC, age=24)
    println(people.maxBy(Person::getPersonAge))  //Person(name=leavesC, age=24)
    println(people.maxBy(Person::filterAge))  //Person(name=leavesC, age=24)
}
```

不管引用的是函数还是属性，都不要在成员引用的名称后面加括号

此外，还可以引用顶层函数

```kotlin
fun test() {
    println("test")
}

fun main() {
    val t = ::test
}
```

也可以用构造方法引用存储或者延期执行创建类实例的动作

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val createPerson = ::Person
    val person = createPerson("leavesC", 24)
    println(person)
}
```

# 十八、标准库中的扩展函数

笔记：常用内置函数.md

# 十九、函数操作符

笔记：函数操作符.md

# 二十、异常

kotlin 中异常处理的基本形式和 Java 类似

```kotlin
fun compute(index: Int): Boolean {
    if (index !in 0..10) {
        throw IllegalArgumentException("参数错误")
    }
    return true
}
```

和 Java 不同的是，kotlin 中 throw 结构是一个表达式，可以作为另一个表达式的一部分来使用

例如下面这个例子，如果条件不满足，则将抛出异常，从而导致 status 变量也不会初始化

```kotlin
val status = if (index in 0..10) index else throw IllegalArgumentException("参数错误")
```

此外，在 Java 中对于受检异常必须显式地处理，通过 try/catch 语句捕获异常或者是抛给其调用者来处理。而 kotlin 不区分受检异常和未受检异常，不用指定函数抛出的异常，可以处理也可以不处理异常

在 kotlin 中 ，try 关键字引入了一个表达式，从而可以把表达式的值赋给一个变量。如果一个 try 代码块执行正常，代码块中最后一个表达式就是结果，如果捕获到了一个异常，则相应 catch 代码块中最后一个表达式就是结果

看以下例子，如果 try 表达式包裹的表达式会抛出异常，则返回值为 null ，否则为 true 

```kotlin
fun main() {
    compute(5)   //fun end : true
    compute(100) //fun end : null
}

fun compute(index: Int) {
    val status = try {
        if (index in 0..10) true else throw IllegalArgumentException("参数错误")
    } catch (e: Exception) {
        null
    }
    println("fun end : " + status)
}
```

但是，如果在 catch 语句中使用 return 结束了 compute 函数，则没有任何输出

```kotlin
fun main() {
    compute(5)   //fun end : true
    compute(100) //没有任何输出
}

fun compute(index: Int) {
    val status = try {
        if (index in 0..10) true else throw IllegalArgumentException("参数错误")
    } catch (e: Exception) {
        return
    }
    println("fun end : " + status)
}
```

# 二十一、运算符重载

kotlin 允许为类型提供预定义的操作符实现，这些操作符具有固定的符号表示（例如 + 和 * ）和固定的优先级，通过操作符重载可以将操作符的行为映射到指定的方法。为实现这样的操作符，需要为类提供一个固定名字的成员函数或扩展函数，相应的重载操作符的函数需要用 operator 修饰符标记

## 1、一元操作符

| 操作符 |      函数      |
| ------ | :------------: |
| +a     | a.unaryPlus()  |
| -a     | a.unaryMinus() |
| !a     |    a.not()     |
| a++    |    a.inc()     |
| a--    |    a.dec()     |

## 2、二元操作符

| 操作符  |       函数       |
| ------- | :--------------: |
| a + b   |    a.plus(b)     |
| a - b   |    a.minus(b)    |
| a * b   |    a.times(b)    |
| a / b   |     a.div(b)     |
| a % b   |     a.rem(b)     |
| a..b    |   a.rangeTo(b)   |
| a in b  |  b.contains(a)   |
| a !in b |  !b.contains(a)  |
| a += b  | a.plusAssign(b)  |
| a -= b  | a.minusAssign(b) |
| a *= b  | a.timesAssign(b) |
| a /= b  |  a.divAssign(b)  |
| a %= b  |  a.remAssign(b)  |

## 3、数组操作符

| 操作符               |          函数           |
| -------------------- | :---------------------: |
| a[i]                 |        a.get(i)         |
| a[i, j]              |       a.get(i, j)       |
| a[i_1, ..., i_n]     |  a.get(i_1, ..., i_n)   |
| a[i] = b             |       a.set(i, b)       |
| a[i, j] = b          |     a.set(i, j, b)      |
| a[i_1, ..., i_n] = b | a.set(i_1, ..., i_n, b) |

## 4、等于操作符

| 操作符 |             函数              |
| ------ | :---------------------------: |
| a == b |  a?.equals(b) ?: b === null   |
| a != b | !(a?.equals(b) ?: b === null) |

相等操作符有一点不同，为了达到正确合适的相等检查做了更复杂的转换，因为要得到一个确切的函数结构比较，不仅仅是指定的名称

方法必须要如下准确地被实现：

```kotlin
operator fun equals(other: Any?): Boolean
```

操作符 ===  和 !==  用来做身份检查（它们分别是 Java 中的 ==  和 !=  ），并且它们不能被重载

## 5、比较操作符

| 操作符 |        函数         |
| ------ | :-----------------: |
| a > b  | a.compareTo(b) > 0  |
| a < b  | a.compareTo(b) < 0  |
| a >= b | a.compareTo(b) >= 0 |
| a <= b | a.compareTo(b) <= 0 |

所有的比较都转换为对  compareTo  的调用，这个函数需要返回  Int  值

## 6、函数调用

| 方法             |          调用           |
| ---------------- | :---------------------: |
| a()              |       a.invoke()        |
| a(i)             |       a.invoke(i)       |
| a(i, j)          |     a.invoke(i, j)      |
| a(i_1, ..., i_n) | a.invoke(i_1, ..., i_n) |

## 7、例子

看几个例子

```kotlin
data class Point(val x: Int, val y: Int) {

    //+Point
    operator fun unaryPlus() = Point(+x, +y)

    //Point++ / ++Point
    operator fun inc() = Point(x + 1, y + 1)

    //Point + Point
    operator fun plus(point: Point) = Point(x + point.x, y + point.y)

    //Point + Int
    operator fun plus(value: Int) = Point(x + value, y + value)

    //Point[index]
    operator fun get(index: Int): Int {
        return when (index) {
            0 -> x
            1 -> y
            else -> throw IndexOutOfBoundsException("无效索引")
        }
    }

    //Point(index)
    operator fun invoke(index: Int) = when (index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("无效索引")
    }

}
```

```kotlin
fun main() {
    //+Point(x=10, y=-20)  =  Point(x=10, y=-20)
    println("+${Point(10, -20)}  =  ${+Point(10, -20)}")

    //Point(x=10, y=-20)++  =  Point(x=10, y=-20)
    var point = Point(10, -20)
    println("${Point(10, -20)}++  =  ${point++}")

    //++Point(x=10, y=-20)  =  Point(x=11, y=-19)
    point = Point(10, -20)
    println("++${Point(10, -20)}  =  ${++point}")

    //Point(x=10, y=-20) + Point(x=10, y=-20)  =  Point(x=20, y=-40)
    println("${Point(10, -20)} + ${Point(10, -20)}  =  ${Point(10, -20) + Point(10, -20)}")

    //Point(x=10, y=-20) + 5  =  Point(x=15, y=-15)
    println("${Point(10, -20)} + ${5}  =  ${Point(10, -20) + 5}")

    point = Point(10, -20)
    //point[0] value is: 10
    println("point[0] value is: ${point[0]}")
    //point[1] value is: -20
    println("point[1] value is: ${point[1]}")

    //point(0) values is: 10
    println("point(0) values is: ${point(0)}")
}
```

# 二十二、中缀调用与解构声明

## 1、中缀调用

可以以以下形式创建一个 Map 变量

```kotlin
fun main() {
    val maps = mapOf(1 to "leavesC", 2 to "ye", 3 to "https://juejin.cn/user/923245496518439")
    maps.forEach { key, value -> println("key is : $key , value is : $value") }
}
```

使用 “to” 来声明 map 的 key 与 value 之间的对应关系，这种形式的函数调用被称为中缀调用

kotlin 标准库中对 to 函数的声明如下所示，其作为扩展函数存在，且是一个泛型函数，返回值 Pair 最终再通过解构声明分别将 key 和 value 传给 Map

```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

中缀调用只能与只有一个参数的函数一起使用，无论是普通的函数还是扩展函数。中缀符号需要通过 **infix** 修饰符来进行标记

```kotlin
fun main() {
    val pair = 10 test "leavesC"
    val pair2 = 1.2 test 20
    println(pair2.javaClass) //class kotlin.Pair
}

infix fun Any.test(other: Any) = Pair(this, other)
```

对于 `mapOf` 函数来说，它可以接收不定数量的 `Pair` 类型对象，因此我们也可以通过自定义的中缀调用符 `test` 来创建一个 map 变量

```kotlin
public fun <K, V> mapOf(vararg pairs: Pair<K, V>): Map<K, V> =
    if (pairs.size > 0) pairs.toMap(LinkedHashMap(mapCapacity(pairs.size))) else emptyMap()

 val map = mapOf(10 test "leavesC", 20 test "hello")
```

## 2、解构声明

有时会有把一个对象拆解成多个变量的需求，在 kotlin 中这种语法称为解构声明

例如，以下例子将 Person 变量结构为了两个新变量：name 和 age，并且可以独立使用它们

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val (name, age) = Person("leavesC", 24)
    println("Name: $name , age: $age")
    //Name: leavesC , age: 24
}
```

一个解构声明会被编译成以下代码：

```kotlin
val name = person.component1()
val age = person.component2()
```

其中的 `component1()` 和 `component2()` 函数是在 kotlin 中广泛使用的约定原则的另一个例子。任何表达式都可以出现在解构声明的右侧，只要可以对它调用所需数量的 `component` 函数即可

需要注意的是，`componentN()` 函数需要用 `operator` 关键字标记，以允许在解构声明中使用它们

对于数据类来说，其自动生成了 `componentN()` 函数，而对非数据类，为了使用解构声明，需要我们自己来手动声明函数

```kotlin
class Point(val x: Int, val y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}

fun main() {
    val point = Point(100, 200)
    val (x, y) = point
    println("x: $x , y: $y")
    //x: 100 , y: 200
}
```

如果我们需要从一个函数返回两个或者更多的值，这时候使用解构声明就会比较方便了

这里使用的是标准类 Pair 来包装要传递的数据，当然，也可以自定义数据类

```kotlin
fun computer(): Pair<String, Int> {
    //各种计算
    return Pair("leavesC", 24)
}

fun main() {
    val (name, age) = computer()
    println("Name: $name , age: $age")
}
```

此外，解构声明也可以用在 for 循环中

```kotlin
val list = listOf(Person("leavesC", 24), Person("leavesC", 25))
for ((name, age) in list) {
    println("Name: $name , age: $age")
}
```

对于遍历 map 同样适用

```kotlin
val map = mapOf("leavesC" to 24, "ye" to 25)
for ((name, age) in map) {
    println("Name: $name , age: $age")
}
```

同样也适用于 lambda 表达式

```kotlin
val map = mapOf("leavesC" to 24, "ye" to 25)
map.mapKeys { (key, value) -> println("key : $key , value : $value") }
```

如果在解构声明中不需要某个变量，那么可以用下划线取代其名称，此时不会调用相应的 `componentN()` 操作符函数

```kotlin
val map = mapOf("leavesC" to 24, "ye" to 25)
for ((_, age) in map) {
    println("age: $age")
}
```

# 二十四、委托

笔记：委托

# 二十五、注解

注解是将元数据附加到代码元素上的一种方式，附加的元数据可以在编译后的类文件或者运行时被相关的源代码工具访问

注解的语法格式如下所示：

```kotlin
annotation class AnnotationName()
```

注解的附加属性可以通过用元注解标注注解类来指定：

- @Target            指定该注解标注的允许范围（类、函数、属性等）
- @Retention         指定该注解是否要存储在编译后的 class 文件中，如果要保存，则在运行时可以通过反射来获取到该注解值
- @Repeatable        标明允许在单个元素上多次使用相同的该注解
- @MustBeDocumented  指定该注解是公有 API 的一部分，并且应该包含在生成的 API 文档中显示的类或方法的签名中

```kotlin
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.FIELD)
@Retention(AnnotationRetention.RUNTIME)
@Repeatable
@MustBeDocumented
annotation class AnnotationName()
```

注解可以声明包含有参数的构造函数

```kotlin
annotation class OnClick(val viewId: Long)
```

允许的参数类型有：

- 原生数据类型，对应 Java 原生的 int 、long、char 等
- 字符串
- class 对象
- 枚举
- 其他注解
- 以上类型的数组

注解参数不能包含有可空类型，因为 JVM 不支持将 null 作为注解属性的值来存储

看一个在运行时获取注解值的例子

```kotlin
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.FIELD)
@Retention(AnnotationRetention.RUNTIME)
annotation class OnClick(val viewId: Long)

class AnnotationsTest {

    @OnClick(200300)
    fun onClickButton() {
        println("Clicked")
    }

}

fun main() {
    val annotationsTest = AnnotationsTest()
    for (method in annotationsTest.javaClass.methods) {
        for (annotation in method.annotations) {
            if (annotation is OnClick) {
                println("method name: " + method.name)  //method name: onClickButton
                println("OnClick viewId: " + annotation.viewId)  //OnClick viewId: 200300
            }
        }
    }
}
```



# 参考

[Android：这是一份全面 & 详细的kotlin入门学习指南（含介绍、配置 & 语法使用）](https://www.jianshu.com/p/e42a9ea09ac0)

[Carson带你学Android：这是一份全面 & 详细的kotlin入门语法指南（类、变量 & 函数）](https://www.jianshu.com/p/d08efdb01bcb)

[Kotlin那些实用的语法糖：空安全、类型转换 & 相等性判断](https://www.jianshu.com/p/37133461ae6b)

[Kotlin：那些关于 类使用 的入门语法都在这里了！](https://www.jianshu.com/p/f447a1ce8ffe)

[两万六千字带你 Kotlin 入门](https://juejin.cn/post/6880602489297895438)
