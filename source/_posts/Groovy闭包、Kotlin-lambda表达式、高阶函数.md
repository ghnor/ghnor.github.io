---
moreLoc: 11
path: 'Others/Groovy闭包、Kotlin lambda表达式、高阶函数.md'
title: 'Groovy闭包、Kotlin lambda表达式、高阶函数'
date: 2018-09-14T22:29:42.000Z
updated: 2018-09-14T22:29:42.000Z
tags:
    - Kotlin
    - Groovy
categories:
    - Others
---

写这篇，主要因为我本身是Android的程序员，在学习Groovy和Kotlin的过程中，对Groovy的闭包，Kotlin的lambda表达式，高阶函数这部分，总是没有直观的概念，直觉上它们还是很magic的东西。所以尝试把它们放到一起，辩证地理解记忆这些概念和语法。

## 没错，它们都是对象

首先Groovy闭包和Kotlin Lambda表达式写法都一样：

```
{parameters -> statements}
```

`->`关键字前面的parameters是参数，后面的statements是表达式。

<!--more-->


Groovy的闭包的是Closure类的实例，如果类型需要强制声明，那么定义Groovy闭包的写法就从：

```Groovy
def groovyClosure = { x -> println(x) }
```

变成了：

```Groovy
Closure groovyClosure = { x -> println(x) }
```

Kotlin的lambda表达式是一组`函数类型`的实例。一组`函数类型`就是比如`() -> Unit`、`(String, Int) -> Boolean`这种。

同样地，如果类型需要强制声明，定义Kotlin Lambda表达式的写法就从：

```Kotlin
var kotlinClosure = { x: String -> println(x) }
```

变成了：

```Kotlin
var kotlinClosure: () -> Unit = { x: String -> println(x) }
```

在Kotlin中，`函数类型`的实例有两类（其实不止，但这里只先说这两类）：

1. Lambda 表达式: `{ a, b -> a + b }`

2. 匿名函数(Anonymous Function): `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

    定义正经函数是这么写的：

    ```Kotlin
    fun kotlinMethod(x: String) {
        println(x)
    }
    ```

    把函数名作为变量名，把函数表达式赋值给该变量，就是匿名函数：

    ```Kotlin
    var kotlinMethod = fun (x: Int) {
        println(x)
    }
    ```

    二者写法在功能性上是完全等价的。当然，也有一些其他区别，因为是变量，所以就算方法是支持重载的，改成匿名函数后，函数参数有变化，也不能取一样的名字了。

在Groovy中，闭包（Closure）的实例只有一类：

1. `{ a, b -> a + b }`

    我就是故意跟上面Kotlin的Lambda表达式写的一样，这俩狗日的其实就是一样的。

## 函数字面量

闭包、lambda表达式，有一万种说法，在不同的语言中，类似的写法类似的功能，有的叫闭包（Closure），有的叫lambda表达式，有的叫块（Block）。

我一直对其非常迷惑，在查资料的过程中，又发现了一种新名词——函数字面量。

在函数式编程语言中，函数就跟变量一样，有其类型和值，是某个类的实例化对象。函数的值就是函数字面量。

同样地，常见的布尔值就是布尔字面量，整数值就是整数字面量，浮点数值就是浮点数字面量。

上面的groovyClosure和kotlinClosure变量被赋值了一段代码块，就是函数字面量。

有一个说法：函数字面量是实现高阶函数，即实现函数参数的一个手段。

## 高阶函数

接受其他函数作为参数，或者返回一个函数，这样的函数称为高阶函数。

这样的解释容易引起误解，因为JS的高阶函数确实将函数作为参数，Kotin也勉强可以这么理解，毕竟能够接受匿名函数作为参数，但是Groovy就不对，Groovy接受的是闭包。

换一个说法：接受其他函数字面量作为参数，或者返回函数字面量，称其为高阶函数。

那么将变量作为函数的参数就是理所当然的事情了。

先看Groovy：

```Groovy
def groovy(closure) {
    closure()
}
```

```Groovy
// 使用已定义的闭包
groovy(groovyClosure)

// 直接写闭包
groovy({ println("Hello Groovy!") })
```

再看Kotlin：

```Kotlin
var kotlin(x: () -> Unit) {
    x()
}
```

```Kotlin
// 使用已定义的函数类型
kotlin(kotlinClosure) // kotlinClosure是Lambda表达式
kotlin(kotlinMethod) // kotlinMethod是匿名函数

// 直接写函数类型
kotlin({ println("Hello Kotlin!") })
```

**有意思的写法**

Groovy和Kotlin的高阶函数都支持一种有意思的写法，如果函数的最后一个参数是闭包或者Lambda的话，可以省略括号。所以调用高阶函数的写法可以变成：

**Groovy：**

```Groovy
groovy { println ("Hello Groovy!") }
```

**Kotlin：**

```Kotlin
kotlin { println("Hello Kotlin!") }
```

## 最后

如果说之前上文所述的很多概念在我脑海中还是各自为锯的知识孤岛，那么现在起码我可以把它们串联起来。尽管有很多的细枝末节还需要深入学习之后才能完善。

（其实就在瞎jiba写）

Over.
