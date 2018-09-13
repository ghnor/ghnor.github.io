写这篇，主要因为我本身是Android的程序员，在学习Groovy和Kotlin的过程中，对Groovy的闭包，Kotlin的lambda表达式，高阶函数这部分，总是没有直观的理解。

所以这里辩证地去看两门语言

## 定义一个函数（方法）

## 函数（方法）

### 函数（方法）定义

**Groovy：**

```Groovy
def groovyMethod(x) {
    println(x)
}
```

**Kotlin：**

```Kotlin
fun kotlinMethod(x: String) {
    println(x)
}
```

### 函数调用

## 定义一个闭包

**Groovy：**

```Groovy
def groovyClosure = { x -> println(x) }
```

**Kotlin：**

```Kotlin
var kotlinClosure = { x: String -> println(x) }
```

## 调用闭包

**Groovy：**

```Groovy
groovyClosure("Groovy")
groovyClosure.call("Groovy")
// 输出：Groovy
```

**Kotlin：**

```Kotlin
kotlinClosure("Kotlin")
// 输出：Kotlin
```

## 函数字面量

闭包、lambda表达式，有一万种说法，在不同的语言中，类似的写法类似的功能，有的叫闭包（Closure），有的叫lambda表达式，有的叫块（Block）。

我一直对其非常迷惑，在查资料的过程中，又发现了一种新名词——函数字面量。

在函数式编程语言中，函数就跟变量一样，有其类型和值，是某个类的实例化对象。函数的值就是函数字面量。

有一个说法：函数字面量是实现高阶函数，即实现函数参数的一个手段。

Groovy的闭包的是Closure类的实例，如果类型需要强制声明，那么上面Groovy的例子就从：

```Groovy
def groovyClosure = { x -> println(x) }
```

变成了：

```Groovy
Closure groovyClosure = { x -> println(x) }
```

Kotlin的lambda表达式就是一组`函数类型`的实例。

一组`函数类型`就是比如`() -> Unit`、`(String, Int) -> Boolean`这种。

同样地，如果类型需要强制声明，上面Kotlin的例子就从：

```Kotlin
var kotlinClosure = { x: String -> println(x) }
```

变成了：

```Kotlin
var kotlinClosure: () -> Unit = { x: String -> println(x) }
```

上面的groovyClosure和kotlinClosure变量被赋值了一段代码块，这一段代码块就是函数字面量。

在Kotlin中，`函数类型`的实例有两类（其实不止，但这里就说这两种）：

1. Lambda 表达式: `{ a, b -> a + b }`

2. 匿名函数(Anonymous Function): `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

    匿名函数可以把函数名作为变量名，把函数表达式赋值给该变量，上文Kotlin函数定义部分的函数完全可以修改成这样：

    ```Kotlin kotlinMethod = fun (x: Int) {
        println(x)
    }
    ```

    跟把函数名字放在`fun`关键字后面的写法是完全等价的。

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

```Groovy
// 使用已定义的闭包
groovy(groovyClosure)
groovy groovyClosure // 省略括号

// 直接写闭包
groovy({ println "Hello Groovy!" })
groovy { println "Hello Groovy!" } // 省略括号
```

再看Kotlin：

```Kotlin
var kotlin(x: () -> Unit) {
    x()
}
```

```Kotlin
kotlin(kotlinMethod) // kotlinMethod是匿名函数

kotlin({ println("Hello Kotlin!") })
```

## 最后

写的很乱，其一，因为本身我没有弄清楚所谓闭包、Lambda表达式这种概念，到底是只是不同语言单纯说法不同，还是有概念的区别。甚至，因为遍历语言发展的过程，这些概念也一直在变化。

其二，在两门语言对比学习的过程中，很难去把握比较的程度。闭包之于Groovy，Lambda表达式之于Kotlin，其实有很多细微的区别，比如有的能访问外部参数，有的不能，有的能直接修改外部变量，有的只是对外部变量的clone。对于函数，Groovy不支持可变数量的参数，而Kotlin可以。再者对于高阶函数，Groovy只能传递闭包，而Kotlin就可以传递匿名函数和Lambda表达式。很多的点，需要在