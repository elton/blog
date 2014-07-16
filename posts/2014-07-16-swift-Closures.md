---
layout: post
title: 'Swift之闭包'
description: '学习Swift闭包的心得'
date: 2014-07-14
comments: true
categories:
- Mac
tags:
- Swift 
- Closures

---
![Swift programming language](https://devimages.apple.com.edgekey.net/home/images/home-hero-swift-hero.png)


[Swift][1] 是一门由[Apple][2] 公司开发的用于iOS和OSX设备上的开发语言，吸收了很多现代开发语言的优势。 今天看了官方的关于闭包部分的文档，感觉很不错，记录一下。

   [1]: https://developer.apple.com/swift/
   [2]: https://www.apple.com
   
闭包是自包含的函数代码块，可以在代码中被传递和使用。 Swift 中的闭包与 C 和 Objective-C 中的代码块（blocks）以及其他一些编程语言中的 lambdas 函数比较相似。
 
闭包可以捕获和存储其所在上下文中任意常量和变量的引用。这就是所谓的闭合并包裹着这些常量和变量，俗称闭包。Swift 会为您管理在捕获过程中涉及到的所有内存操作。

Swift 的闭包表达式拥有简洁的风格，并鼓励在常见场景中进行语法优化，主要优化如下：

 * 利用上下文推断参数和返回值类型
 * 隐式返回单表达式闭包，即单表达式闭包可以省略return关键字
 * 参数名称缩写
 * 尾随（Trailing）闭包语法



### 举例
如下面例子，把数组中的String排序

```
let names = ["Chris","Alex", "Ewa", "Barry", "Daniella"]
```
使用sorted函数

```
func backwards(s1: String, s2: String)-> Bool {  
   return s1 > s2  
}  
var reversed = sorted(names, backwards)  
// reversed 为 ["Ewa","Daniella", "Chris", "Barry", "Alex"]  
```
下面开始逐步简化了


###### 使用闭包表达式（Closure Expression）
语法形式如下：

```
{ (parameters) -> returnType in  
   statements  
}  
```
则上面的代码可以简化成如下形式：

```
var reversed = sorted(names, {(s1:String, s2:String) -> Bool in return s1 > s2})
```

###### 根据上下文推断类型（Inferring Type From Context）
因为排序闭包函数是作为sort函数的参数进行传入的，Swift可以推断其参数和返回值的类型。 sort期望第二个参数是类型为(String, String) -> Bool的函数，因此实际上String,String和Bool类型并不需要作为闭包表达式定义中的一部分。 因为所有的类型都可以被正确推断，返回箭头 (->) 和围绕在参数周围的括号也可以被省略：

```
var reversed = sorted(names, { s1, s2 in return s1 > s2 })
```
 
##### 单表达式闭包隐式返回（Implicit Return From Single-Expression Clossures）
单行表达式闭包可以通过隐藏return关键字来隐式返回单行表达式的结果，如上版本的例子可以改写为：

```
var reversed = sorted(names, { s1, s2 in s1 > s2 })
```
在这个例子中，sort函数的第二个参数函数类型明确了闭包必须返回一个Bool类型值。因为闭包函数体只包含了一个单一表达式 (s1 > s2)，该表达式返回Bool类型值，因此这里没有歧义，return关键字可以省略。


##### 参数名称缩写（Shorthand Argument Names）
Swift 自动为内联函数提供了参数名称缩写功能，您可以直接通过$0,$1,$2来顺序调用闭包的参数。

如果您在闭包表达式中使用参数名称缩写，您可以在闭包参数列表中省略对其的定义，并且对应参数名称缩写的类型会通过函数类型进行推断。 in关键字也同样可以被省略，因为此时闭包表达式完全由闭包函数体构成：

```
var reversed = sorted(names, { $0 > $1 })
```


##### 运算符函数（Operator Functions）
实际上还有一种更简短的方式来撰写上面例子中的闭包表达式。 Swift 的String类型定义了关于大于号 (>) 的字符串实现，其作为一个函数接受两个String类型的参数并返回Bool类型的值。而这正好与sort函数的第二个参数需要的函数类型相符合。因此，您可以简单地传递一个大于号，Swift可以自动推断出您想使用大于号的字符串函数实现：

```
var reversed = sorted(names, >)
```
怎么样，够简洁吧！


### 尾随闭包（Trailing Closures）
如果您需要将一个很长的闭包表达式作为最后一个参数传递给函数，可以使用尾随闭包来增强函数的可读性。尾随闭包是一个书写在函数括号之后的闭包表达式，函数支持将其作为最后一个参数调用。

```
func someFunctionThatTakesAClosure(closure:() -> ()) {  
   // 函数体部分  
}  
   
// 以下是不使用尾随闭包进行函数调用  
   
someFunctionThatTakesAClosure({  
   // 闭包主体部分  
    })  
   
// 以下是使用尾随闭包进行函数调用  
   
someFunctionThatTakesAClosure() {  
   // 闭包主体部分  
}
```
**注意** 如果函数只需要闭包表达式一个参数，当您使用尾随闭包时，您甚至可以把()省略掉。

在上例中作为sort函数参数的字符串排序闭包可以改写为：

```
var reversed = sorted(names) { $0 > $1 }
```

__当闭包非常长以至于不能在一行中进行书写时，尾随闭包变得非常有用__

```
let digitNames = [
    0: "Zero", 1: "One", 2: "Two",   3: "Three", 4: "Four",
    5: "Five", 6: "Six", 7: "Seven", 8:"Eight", 9: "Nine"
]
let numbers = [16, 58, 510]

let strings = numbers.map {
    (var number) -> String in
    var output = ""
    while number > 0 {
        output = digitNames[number%10]! + output
        number /= 10
    }
    return output
}
// strings 常量被推断为字符串类型数组，即String[]  
// 其值为["OneSix", "FiveEight", "FiveOneZero"]  
```

### 捕获值（Capturing Values）
闭包可以在其定义的上下文中捕获常量或变量。即使定义这些常量和变量的原域已经不存在，闭包仍然可以在闭包函数体内引用和修改这些值。
 
Swift最简单的闭包形式是嵌套函数，也就是定义在其他函数的函数体内的函数。 嵌套函数可以捕获其外部函数所有的参数以及定义的常量和变量。
 
下例为一个叫做makeIncrementor的函数，其包含了一个叫做incrementor嵌套函数。嵌套函数incrementor从上下文中捕获了两个值，runningTotal和amount。之后makeIncrementor将incrementor作为闭包返回。每次调用incrementor时，其会以amount作为增量增加runningTotal的值。

```
func makeIncrementor(forIncrement amount:Int) -> () -> Int {  
   var runningTotal = 0  
   func incrementor() -> Int {  
       runningTotal += amount  
       return runningTotal  
    }  
   return incrementor  
} 

let incrementByTen =makeIncrementor(forIncrement: 10)  
incrementByTen()  
// 返回的值为10  
incrementByTen()  
// 返回的值为20  
incrementByTen()  
// 返回的值为30 

let incrementBySeven =makeIncrementor(forIncrement: 7)  
incrementBySeven()  
// 返回的值为7  
incrementByTen()  
// 返回的值为40 
```


### 闭包是引用类型（Closures Are Reference Types）
上面的例子中，incrementBySeven和incrementByTen是常量，但是这些常量指向的闭包仍然可以增加其捕获的变量值。这是因为函数和闭包都是引用类型。

```
let alsoIncrementByTen = incrementByTen  
alsoIncrementByTen()  
// 返回的值为50  
```





