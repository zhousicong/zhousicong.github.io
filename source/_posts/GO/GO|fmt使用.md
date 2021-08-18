---
title: Go|fmt使用
categories:
  - GO
abbrlink: 2dd94f98
date: 2021-08-18 17:29:02
description:
tags:
---
fmt是Go中最常用的包之一，主要实现了格式化I/O(输入/输出)。
<!-- more -->
fmt包大致分为输出和输入的两大部分。

### **输出**
输出部分包括三个系列和一个独立的函数：Print系列，Fprint系列，Sprint系列，以及Errorf()。这些函数的使用场景如下：
- 向终端输出一些信息的时候，使用Print系列。（使用程度：频繁）
- 将信息写入文件中时，使用Fprint系列。（使用程度：一般）
- 在程序中获取格式化字符串中时，使用Sprint系列。（使用程度：一般）
- 在程序中获取包含格式化字符串的错误时，使用Errorf()。（使用程度：几乎不）

### Print系列
Print系列中包含三个重要的函数：Print()，Printf()，Println()。
```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```
Print()将参数的内容进行标准输出，而Println()则是在标准输出后进行换行。（Println()中的ln意思为line）
让我们来测试一下这两个函数。第一次展示程序的所有内容，后面则会省略。
```go
package main

import "fmt"

func main() {
    var a string = "test Println"
    fmt.Println(a)
    var b string = "test Print"
    fmt.Print(b)
}
```
终端输出的结果为：
```go
test Println   //换行
test Print
```     
再来看一下Printf()。Printf()用于格式化字符串的输出。举个例子：
```go
var date string = "20200408"
fmt.Printf("今天的日期是：%s", date)
``` 
注意，这里的%s代表字符串的占位符，目的是为了告诉程序将date变量的代入位置。计算结果：
```
今天的日期是：20200408
```

### Fprint系列
Fprint系列包含三个函数：Fprint(),Fprintf(),Fprintln()。
```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```
Fprint系列与Print系列相比多了一个io.Writer接口类型的参数w。Fprint系列函数会将内容输出到参数w中。只要参数类型实现了io.Writer接口，则都可以实现写入。
```go
// 打开xx.txt文件
fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
    fmt.Println("打开文件错误：err:", err)
    return
}
s := "test"
// 向打开的文件中写入格式化字符串
fmt.Fprintf(fileObj, "往文件中写如信息：%s", s)
```
实际上，Print系列其实就是通过封装了Fprint系列来实现的。Print()的源代码如下：
```go
func Print(a ...interface{}) (n int, err error) {
    return Fprint(os.Stdout, a...)
}
```
调用Print()后返回了一个Fprint()，而os.Stdout代表标准输出。因此我们可以用Fprint()来实现与Print()。
```go
var a string = "test Fprint"
fmt.Fprintln(os.Stdout, a)

//output
//test Fprint
```

### Sprint系列
Sprint系列函数会把传入的参数生成并返回一个字符串。
Sprint系列包含三个函数：Sprint(),Sprintf(),Sprintln()。
```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```
Sprint系列与Print系列的区别在于输出的对象不同，Sprint系列的输出对象为字符串。利用time包，举一个输出日期和时间的例子：
```go
year, month, day := time.Now().Date()
hour, min, sec := time.Now().Clock()
//将格式化字符串写入变量s1中
s1 := fmt.Sprintf("今天的日期的是：%d年%d月%d日，现在的时间是：%d:%d:%d\n",year,month,day,hour,min,sec)
fmt.Println(s1)

//output
//今天的日期的是：2020年4月8日，现在的时间是：22:23:39
```

### Errorf函数
Errorf()根据format参数生成格式化字符串并返回一个包含该字符串的错误。
```go
func Errorf(format string, a ...interface{}) error
```
举个例子
```go
error := "未知"
err := fmt.Errorf("这个错误类型为：%s", error)
fmt.Println(err)

//output
//这个错误类型为：未知
```
它的底层是通过error包的new()中传入Sprintf()来实现的：
```go
func Errorf(format string, a ...interface{}) error {
    return errors.New(Sprintf(format, a...))
}
```
这也就是为什么Errorf()没有它的兄弟Error()和Errorln()的原因了。因为我们可以直接通过errors.New()来生成一个非格式化字符串的错误。