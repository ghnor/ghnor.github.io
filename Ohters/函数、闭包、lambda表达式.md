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



闭包、lambda表达式，有一万种说法，在不同的语言中，类似的写法类似的功能，有的叫闭包（Closure），有的叫lambda表达时候，有的叫块（Block）。

我一直对其非常，在查资料的过程中，又发现了一种新名词——函数字面量。

我理解它是把函数表达式赋值给一个变量，也就是把函数表达式声明为某种类型的实例，这个变量就是函数字面量。

比如，Groovy的闭包的是Closure类的实例，如果类型需要强制声明，那么上面的例子就从

```Groovy
def groovy = { println "Hello Groovy!" }
```

变成了

```Groovy
Closure groovy = { println "Hello Groovy!" }
```

同样的，换成Kotlin的话，lambda表达式就是一组`函数类型`的实例。

一组`函数类型`就是比如`() -> Unit`、`(String, Int) -> Boolean`这种。

同样地，如果类型需要强制声明，上面Kotlin的例子就从：

```Kotlin
var kotlin = { println("Hello Kotlin!") }
```

变成了：

```Kotlin
var kotlin: () -> Unit = { println("Hello Kotlin!") }
```

上面的groovy和kotlin变量被赋值了一段代码块，这就是函数字面量。

在Kotlin中，`函数类型`的实例有两类：

1. Lambda 表达式: `{ a, b -> a + b }`

2. 匿名函数(Anonymous Function): `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

在Groovy中，闭包（Closure）的实例只有一类：

1. `{ a, b -> a + b }`

    我就是故意跟上面Kotlin的Lambda表达式写的一样，这俩狗日的其实就是一样的。

2. 不好意思，Groovy的匿名函数跟

写到这里，主要想表达的是，一个闭包跟一个普通函数特别类似，感觉只有语法的区别。闭包的关键字是`->`，而函数的关键字是`fun`；闭包把参数放在`{}`里面，用`->`分隔，而函数把参数放在`{}`外面，用`()`包裹。

##

继续看高阶函数，主要这里