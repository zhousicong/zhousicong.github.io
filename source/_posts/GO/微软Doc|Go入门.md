---
title: 微软Doc|Go入门
categories:
  - GO
tags:
  - GO教程
abbrlink: eb7fce26
date: 2021-08-26 17:26:43
description:
---
[微软-Go入门](https://docs.microsoft.com/zh-cn/learn/paths/go-first-steps/)

<!-- more -->
<details>
<summary>目录</summary>

- [声明和使用变量](#声明和使用变量)
- [初始化变量](#初始化变量)
- [通过多种方式初始化变量](#通过多种方式初始化变量)
- [声明常量](#声明常量)
- [基本数据类型](#基本数据类型)
  - [整数数字](#整数数字)
  - [浮点数字](#浮点数字)
  - [布尔型](#布尔型)
  - [字符串](#字符串)
  - [默认值](#默认值)
- [函数](#函数)
  - [main函数](#main函数)
  - [自定义函数](#自定义函数)
  - [更改函数参数值(指针)](#更改函数参数值指针)
- [在GO中使用控制流](#在go中使用控制流)
  - [defer函数](#defer函数)
  - [panic函数](#panic函数)
- [数据类型](#数据类型)
  - [数组](#数组)
  - [切片](#切片)
  - [映射](#映射)
- [goroutine](#goroutine)
- [channel](#channel)
  - [无缓冲 channel 与有缓冲 channel](#无缓冲-channel-与有缓冲-channel)
</details>
### 声明和使用变量
```go
// 声明单个变量
var firstName string
// 声明多个变量
var firstName, lastName string
// 声明不同类型变量
var firstName, lastName string
var age int
// or
var (
    firstName, lastName string
    age int
)
```

### 初始化变量
```go
// 指定类型初始化变量
var (
    firstName string = "John"
    lastName  string = "Doe"
    age       int    = 32
)
// 不指定类型初始化变量，Go会推断出类型
var (
    firstName = "John"
    lastName  = "Doe"
    age       = 32
)
```
### 通过多种方式初始化变量
在 Go 中，你可以在单行中声明和初始化变量。 使用逗号将每个变量名称隔开，并对每个值执行相同的操作（按同一顺序），如下所示：
```go
var (
    firstName, lastName, age = "John", "Doe", 32
)
```
还可以通过另一种方式来声明和初始化变量。 此方法是在 Go 中执行此操作的最常见方法。 以下是我们使用的同一个示例说明：
```go
func main() {
    firstName, lastName := "John", "Doe"
    age := 32
    println(firstName, lastName, age)
}
```
运行上述代码，确认此方法能否声明和初始化变量。

请注意，在定义变量名称后，需要在此处加入一个冒号等于号 (:=) 和相应的值。 使用冒号等于号时，要声明的变量必须是新变量。 如果使用冒号等于号并已经声明该变量，将不会对程序进行编译。 继续尝试。

最终，你能在函数内使用冒号等于号。 在声明函数外的变量时，必须使用 var 关键字执行此操作。 如果你不熟悉函数，请不要担心。 我们会在后续单元中介绍函数。

### 声明常量
```go
const HTTPStatusOK = 200
```

### 基本数据类型
Go 是一种强类型语言。 这意味着你声明的每个变量都绑定到特定的数据类型，并且只接受与此类型匹配的值。
Go 有四类数据类型：
- 基本类型：数字、字符串和布尔值
- 聚合类型：数组和结构
- 引用类型：指针、切片、映射、函数和通道
- 接口类型：接口

#### 整数数字
- int、int8、int16、int32 和 int64 类型,其大小分别为 8、16、32 或 64 位的整数。
- uint、uint8、uint16、uint32 和 uint64 类型
#### 浮点数字
- float32、fload64
#### 布尔型
- true、false
#### 字符串
字符串转义符
- \n：新行
- \r：回车符
- \t：选项卡
- \'：单引号
- \"：双引号
- \\：反斜杠
#### 默认值
- int 类型的 0（及其所有子类型，如 int64）
- float32 和 float64 类型的 +0.000000e+000
- bool 类型的 false
- string 类型的空值
### 函数
#### main函数
Go中所有可执行程序都具有此函数，因为他是程序的起点。
#### 自定义函数
```go
// 示例
func name(parameters) (results) {
    body-content
}

func sum(number1 string, number2 string) (result int) {
    int1, _ := strconv.Atoi(number1)
    int2, _ := strconv.Atoi(number2)
    result = int1 + int2
    return
}
// 返回多个值
func calc(number1 string, number2 string) (sum int, mul int) {
    int1, _ := strconv.Atoi(number1)
    int2, _ := strconv.Atoi(number2)
    sum = int1 + int2
    mul = int1 * int2
    return
}
```
#### 更改函数参数值(指针)
将值传递给函数时，该函数中的每个更改都不会影响调用方。 Go 是“按值传递”编程语言。 这意味着每次向函数传递值时，Go 都会使用该值并创建本地副本（内存中的新变量）。 在函数中对该变量所做的更改都不会影响你向函数发送的更改。
```go
package main

func main() {
    firstName := "John"
    updateName(firstName)
    println(firstName)
}

func updateName(name string) {
    name = "David"
}
```
即使你在函数中将该名称更改为 David，输出仍为 John。 由于 updateName 函数中的更改仅会修改本地副本，因此输出不会发生变化。 Go 传递变量的值，而不是变量本身。

如果你希望在 updateName 函数中进行的更改会影响 main 函数中的 firstName 变量，则需要使用指针。 指针 是包含另一个变量的内存地址的变量。 当你发送指向某个函数的指针时，不会传递值，而是传递地址内存。 因此，对该变量所做的每个更改都会影响调用方。

在 Go 中，有两个运算符可用于处理指针：

& 运算符使用其后对象的地址。
* 运算符取消引用指针。 也就是说，你可以前往指针中包含的地址访问其中的对象。
`*可以看成是&的逆运算`
让我们修改前面的示例，以阐明指针的工作方式：
```go
package main

func main() {
    firstName := "John"
    updateName(&firstName)
    println(firstName)
}

func updateName(name *string) {
    *name = "David"
}
```
运行前面的代码。 请注意，输出现在显示的是 David，而不是 John。

首先要做的就是修改函数的签名，以指明你要接收指针。 为此，请将参数类型从 string 更改为 *string。 （后者仍是字符串，但现在它是指向字符串 的 指针。）然后，将新值分配给该变量时，需要在该变量的左侧添加星号 (*) 以暂停该变量的值。 调用 updateName 函数时，系统不会发送值，而是发送变量的内存地址。 这就是前面的代码在变量左侧带有 & 符号的原因。
### 在GO中使用控制流
> Go不支持三元if语句(即三元表达式)
> Go 支持if、Switch、for，Go没有while关键字

#### defer函数
在 Go 中，defer 语句会推迟函数（包括任何参数）的运行，直到包含 defer 语句的函数完成。 通常情况下，当你想要避免忘记任务（例如关闭文件或运行清理进程）时，可以推迟某个函数的运行。
可以根据需要推迟任意多个函数。 defer 语句按逆序运行，先运行最后一个，最后运行第一个。
通过运行以下示例代码来查看此模式的工作原理：
```go
package main

import "fmt"

func main() {
    for i := 1; i <= 4; i++ {
        defer fmt.Println("deferred", -i)
        fmt.Println("regular", i)
    }
}
```
下面是代码输出:
```
regular 1
regular 2
regular 3
regular 4
deferred -4
deferred -3
deferred -2
deferred -1
```

#### panic函数
运行时错误会使 Go 程序崩溃，例如尝试通过使用超出范围的索引或取消引用 nil 指针来访问数组。 你也可以强制程序崩溃。
内置 panic() 函数可以停止 Go 程序中的正常控制流。 当你使用 panic 调用时，任何延迟的函数调用都将正常运行。 进程会在堆栈中继续，直到所有函数都返回。 然后，程序会崩溃并记录日志消息。 此消息包含错误信息和堆栈跟踪，有助于诊断问题的根本原因。
调用 panic() 函数时，可以添加任何值作为参数。 通常，你会发送一条错误消息，说明为什么会进入紧急状态。
例如，下面的代码将 panic 和 defer 函数组合在一起。 尝试运行此代码以了解控制流的中断。 请注意，清理过程仍会运行。

### 数据类型
#### 数组
Go 中的数组是一种特定类型且长度固定的数据结构。 它们可具有零个或多个元素，你必须在声明或初始化它们时定义大小。 此外，它们一旦创建，就无法调整大小。 鉴于这些原因，数组在 Go 程序中并不常用，但它们是切片和映射的基础。
```go
// 初始化数据
var a [3]int
cities := [5]string{"New York", "Paris", "Berlin", "Madrid"}
```
**数组中的省略号**
如果你不知道你将需要多少个位置，但知道你将具有多少数据，那么还有一种声明和初始化数组的方法是使用省略号 (...)，如下例所示：
```
q := [...]int{1, 2, 3}
cities := [...]string{"New York", "Paris", "Berlin", "Madrid"}
```
#### 切片
**数组的长度是固定的，切片的大小是动态的，不固定的**
切片运算符`s[i:p]`
表示s中从下标 startIndex 到 endIndex-1 下的元素创建为一个新的切片
**注意，切片只能引用元素的子集**
#### 映射
大体上来说，Go 中的映射是一个哈希表，是键值对的集合。 映射中所有的键都必须具有相同的类型，它们的值也是如此。 不过，可对键和值使用不同的类型。 例如，键可以是数字，值可以是字符串。 若要访问映射中的特定项，可引用该项的键。
```go
studentsAge := map[string]int{
    "john": 32,
    "bob":  31,
}
studentsAge := make(map[string]int)
studentsAge["bob"] = 31
delete(studentsAge, "bob")
for name, age := range studentsAge {
    fmt.Printf("%s\t%d\n", name, age)
}
```
### goroutine
**不是通过共享内存通信，而是通过通信共享内存**
goroutine 是轻量线程中的并发活动，而不是在操作系统中进行的传统活动。 假设你有一个写入输出的程序和另一个计算两个数字相加的函数。 一个并发程序可以有数个 goroutine 同时调用这两个函数。
### channel
#### 无缓冲 channel 与有缓冲 channel
现在，你可能想知道何时使用这两种类型。 这完全取决于你希望 goroutine 之间的通信如何进行。 无缓冲 channel 同步通信。 它们保证每次发送数据时，程序都会被阻止，直到有人从 channel 中读取数据。
相反，有缓冲 channel 将发送和接收操作解耦。 它们不会阻止程序，但你必须小心使用，因为可能最终会导致死锁（如前文所述）。 使用无缓冲 channel 时，可以控制可并发运行的 goroutine 的数量。 例如，你可能要对 API 进行调用，并且想要控制每秒执行的调用次数。 否则，你可能会被阻止。