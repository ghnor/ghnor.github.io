## 定义一个函数（方法）

* return可以省略，默认返回函数最后一行一句的执行结果。

**Groovy：**

```Groovy
def groovy() {
    println "Hello Groovy!"
}

def groovy(x) {
    println x
}
```

**Kotlin：**

```Kotlin
var kotlinMethod = fun () {
    println("Hello Kotlin!")
}

var kotlinMethod1 = fun (x: Int) {
    println(x)
}
```

OR：

```Kotlin
fun kotlinMethod() {
    println("Hello Kotlin!")
}

fun kotlinMethod1(x: String) {
    println(x)
}
```

## 定义一个闭包

**Groovy：**

```Groovy
// 无参
def groovy = { println "Hello Groovy!" }

// 有参
def groovy1 = { x -> println x }
```

**Kotlin：**

```Kotlin
// 无参
var kotlin = { println("Hello Kotlin!") }

// 有参
var kotlin1 = { x: String -> println(x) }
```

## 调用闭包

**Groovy：**

```Groovy
groovy() //输出：Hello Groovy!
groovy.call()

groovy1("Groovy") //输出：Groovy
groovy1.call("Groovy")
groovy1 "Groovy"
```

**Kotlin：**

```Kotlin
kotlin() //输出：Hello Kotlin!

kotlin1("Kotlin") //输出：Kotlin
```

闭包、lambda表达式，有一万种说法，在不同的语言中，类似的写法类似的功能，有的叫闭包（Closure），有的叫lambda表达式，有的叫块（Block）。

我一直对其非常迷惑，在查资料的过程中，又发现了一种新名词——函数字面量。

在函数式编程语言中，函数就跟变量一样，有其类型和值，是某个类的实例化对象。函数的值就是函数字面量。

Groovy的闭包的是Closure类的实例，如果类型需要强制声明，那么上面Groovy的例子就从：

```Groovy
def groovy = { println "Hello Groovy!" }
```

变成了：

```Groovy
Closure groovy = { println "Hello Groovy!" }
```

Kotlin的lambda表达式就是一组`函数类型`的实例。

一组`函数类型`就是比如`() -> Unit`、`(String, Int) -> Boolean`这种。

同样地，如果类型需要强制声明，上面Kotlin的例子就从：

```Kotlin
var kotlin = { println("Hello Kotlin!") }
```

变成了：

```Kotlin
var kotlin: () -> Unit = { println("Hello Kotlin!") }
```

上面的groovy和kotlin变量被赋值了一段代码块，这一段代码块就是函数字面量。

在Kotlin中，`函数类型`的实例有两类（其实不止，但这里就说这两种）：

1. Lambda 表达式: `{ a, b -> a + b }`

2. 匿名函数(Anonymous Function): `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

在Groovy中，闭包（Closure）的实例只有一类：

1. `{ a, b -> a + b }`

    我就是故意跟上面Kotlin的Lambda表达式写的一样，这俩狗日的其实就是一样的。

写到这里，主要想表达的是，一个闭包跟一个普通函数特别类似，感觉只有语法的区别。闭包的关键字是`->`，而函数的关键字是`fun`；闭包把参数放在`{}`里面，用`->`分隔，而函数把参数放在`{}`外面，用`()`包裹。

## 高阶函数

接收其他函数作为参数，或者返回一个函数，这样的函数称为高阶函数。

上面说了，不论是Kotlin中函数类型，还是Groovy中的闭包，都是一种类，Lambda表达式或匿名函数和闭包都是此类的实例化对象。

那么其可以作为函数的参数就是理所当然的事情了。

先看Groovy：

```Groovy
def groovy(closure) {
    closure()
}
```

再看Kotlin：

```Kotlin
var kotlin(x: () -> Unit) {
    x()
}
```