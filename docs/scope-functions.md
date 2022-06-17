---
title: Kotlin常用作用域函数
categories: -技术
tags: 
- Kotlin
- Android
date: 2022-03-18 10:21 
---
## 意义
作用域函数没有引入任何新的技术，但是它们可以使代码更加简洁易读。针对编码场景选择相应的作用域函数。

## 区别
由于作用域函数本质上都非常相似，因此了解它们之间的区别很重要。每个作用域函数之间有两个主要区别：
- 引用上下文对象的方式
- 返回值

## let

**上下文对象**作为 lambda 表达式的参数（`it`）来访问。**返回值**是 lambda 表达式的结果。

1.判断对象是否为空执行后续代码块

```kotlin
var user:User ? = null
user?.let{//it
	print(it.toString())	//当user为空时则不会执行print，反之。
}
```

2.在调用链的结果上调用一个或多个函数。

``` kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let {//it 
    println(it)
    // 如果需要可以调用更多函数
} 
```

3.若代码块仅包含以 `it` 作为参数的单个函数，则可以使用方法引用(`::`)代替 lambda 表达式

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let(::println)
```

4.引入作用域受限的局部变量以提高代码的可读性。如需为上下文对象定义一个新变量，可提供其名称作为 lambda 表达式参数来替默认的 `it`。

```kotlin
val numbers = listOf("one", "two", "three", "four")
val modifiedFirstItem = numbers.first().let { firstItem ->
    println("The first item of the list is '$firstItem'")
    if (firstItem.length >= 5) firstItem else "!" + firstItem + "!"
}.toUpperCase()
println("First item after modifications: '$modifiedFirstItem'")
```

## with

**上下文对象**作为参数传递，但是在 lambda 表达式内部，它可以作为接收者（`this`）使用。 **返回值**是 lambda 表达式结果。

1.`with` 可以理解为“*对于这个对象，执行以下操作。*而不使用lambda表达式结果

``` kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {//numbers作为this参数引入																			
    println("'with' is called with argument $this")	//this（numbers）对象
    println("It contains $size elements")
}
```
2.with 的另一个使用场景是引入一个辅助对象，其属性或函数将用于计算一个值。

```kotlin
val numbers = mutableListOf("one", "two", "three")
val firstAndLast = with(numbers) {
    "The first element is ${first()}," +
    " the last element is ${last()}"
}
println(firstAndLast)			//打印拼装后的字符串
```

## run

**上下文对象** 作为接收者（`this`）来访问。 **返回值** 是 lambda 表达式结果。

`run` 和 `with` 做同样的事情，但是调用方式和 `let` 一样——作为上下文对象的扩展函数.

1.当 lambda 表达式同时包含对象初始化和返回值的计算时，`run` 很有用。

```kotlin
val service = MultiportService("https://example.kotlinlang.org", 80)

val result = service.run {//this
    port = 8080
    query(prepareRequest() + " to port $port")	//返回query 运算结果
}

// 同样的代码如果用 let() 函数来写:
val letResult = service.let {
    it.port = 8080
    it.query(it.prepareRequest() + " to port ${it.port}")	//返回query运算结果
}
```

2.作为非扩展函数使用。 非扩展 `run` 可以在需要表达式的地方执行一个由多个语句组成的块。

```kotlin
val hexNumberRegex = run {//this
  	//定义多个块属性
    val digits = "0-9"
    val hexDigits = "A-Fa-f"
    val sign = "+-"
    Regex("[$sign]?[$digits$hexDigits]+")	//返回填充后的Regex
}

for (match in hexNumberRegex.findAll("+1234 -FFFF not-a-number")) {
    println(match.value)									//打印匹配到的值
}
```

## apply

**上下文对象** 作为接收者（`this`）来访问。 **返回值** 是上下文对象本身。

1.通常apply用作对象配置填充，返回对象本身

```kotlin
val adam = Person("Adam").apply {//this
    age = 32
    city = "London"        
}
println(adam)
```

##  also

**上下文对象**作为 lambda 表达式的参数（`it`）来访问。 **返回值**是上下文对象本身。

`also` 对于执行一些将上下文对象作为参数的操作很有用。 对于需要引用对象而不是其属性与函数的操作，或者不想屏蔽来自外部作用域的 `this` 引用时，请使用 `also`。

`also`跟`apply` 很像但用法完全不同

1.当看到 `also` 时，可以将其理解为“*并且用该对象执行以下操作*”。

```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("The list elements before adding new one: $it") }
    .add("four")
```

## 函数选择

| 函数    | 对象引用 | 返回值            | 是否是扩展函数             |
| :------ | :------- | :---------------- | :------------------------- |
| `let`   | `it`     | Lambda 表达式结果 | 是                         |
| `run`   | `this`   | Lambda 表达式结果 | 是                         |
| `run`   | -        | Lambda 表达式结果 | 不是：调用无需上下文对象   |
| `with`  | `this`   | Lambda 表达式结果 | 不是：把上下文对象当做参数 |
| `apply` | `this`   | 上下文对象        | 是                         |
| `also`  | `it`     | 上下文对象        | 是                         |

- 对一个非空（non-null）对象执行 lambda 表达式：`let`
- 将表达式作为变量引入为局部作用域中：`let`
- 对象配置：`apply`
- 对象配置并且计算结果：`run`
- 在需要表达式的地方运行语句：非扩展的 `run`
- 附加效果：`also`
- 一个对象的一组函数调用：`with`

## 引用

[Kotlin官方文档](https://www.kotlincn.net/docs/reference/scope-functions.html)
[Youtube Philipp Lackner视频教学](https://www.youtube.com/watch?v=Vy-dS2SVoHk&t=393s&ab_channel=PhilippLackner) 

视频教学下最精辟的总结评论

Summary of Scope functions: let: Used to check nulls, also better than simple null check in multi-threading case also: same as 'let'  but it doesn't return the last line as 'let', instead 'also'  will return the object it was called on and 'not the last line!' apply: helpful function to modify objects, if you want to change in properties of the objects, and it uses 'this' instead of 'it' as we work inside the class of the object run: equivalent to 'apply', but it won't return the object it was called, instead it will return the last line  with: same as 'run' but a different signature.

