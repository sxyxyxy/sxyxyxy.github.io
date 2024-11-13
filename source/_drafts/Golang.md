---
title: Golang
date: 2024-11-11 22:00:00
description: Golang笔记
categories:
  - Programming Language
tags: 
  - Golang
---
# Golang笔记
此笔记基于[Go Programming – Golang Course with Bonus Projects](https://www.youtube.com/watch?v=un6ZyFkqFKo)视频和[Go语言圣经](https://gopl-zh.github.io/)。
Go是一种编译型语言，有着比解释型语言更快的执行速度和比一些其他编译型语言（如C++,rust）更快的编译速度。Go有Garbage Collection机制，能够自动回收没用的变量空间。

## Hello world

```Go
package main

import "fmt"

func main() {
    fmt.Println("hello world")
}
```

每个Go源文件开始都使用package声明这个源文件属于那个包，后面的import导入需要用到的模块（导入没有使用的模块会发生编译错误，圆括号能导入多个包），fmt包用来格式化输出和扫描输入。

## 程序结构
### 声明
Go语言中的函数名,变量名,常量名,类型名,语句标号和包名等所有的命名，都以字母（unicode）或下划线开头，后面跟任意数量的字母、数字和下划线，**区分大小写**，通常采用**驼峰式**命名。
对于包级别的声明首字母的大小写决定它是否是私有的（大写包外也能访问，小写私有）。
Go语言主要有4种类型声明语句：var（变量）、const（常量）、type（类型）、func（函数）。
```Go
package main

import "fmt"

const boilingF = 212.0		// 常量声明

func main() {			// 函数声明
	var f = boilingF	// 变量声明
	var c = (f - 32) * 5 / 9
	fmt.Printf("boiling point = %gF or %g°C\n", f, c)
	// boiling point = 212°F or 100°C
}
```

### 变量声明
Go变量声明的一般方法为：`var 名称 类型 = 表达式`
类型和表达式可以省略其中的一个,对于省略类型的变量，Go会根据赋予的初值自动决定变量的类型。对于省略了表达式的变量会被默认初始化为该类型所对应的零值（数值为0，bool为false，string为""，接口或引用类型为nil，数组和结构体为每个字段为对应零值的值）。Go如果存在声明却未使用的变量会发生编译错误。
还有一种简短变量声明的方法，使用`名称 := 表达式`的形式声明变量。在使用简短变量声明声明多个变量时，如果名称中有已经存在的变量，则对于该变量该操作相当于赋值，使用简短变量声明时必须至少声明一个新变量，否则会发生编译错误。
```Go
f, err := os.Open(infile)
// ...
f, err := os.Create(outfile)	// compile error: no new variable
// 第二个声明应该为赋值
```
```Go
var number int
var number int = 123                
var number = 123                    
number := 123
// 也可以同时声明多个变量
var a, b, c int                     // int, int, int 只能同类型
var (					// 可以不同类型
	x int
	y float
	z string
) 
var d, e, f = true, 2, "abc"        // bool int string 
g, h := 123, "aaa"                  // int string
// 还能使用new(T)函数声明变量，返回*T指针
p := new(int)		// 是一种语法糖，除了没有变量名称和普通变量没什么区别
fmt.println(*p)		// 0
```

### 指针
每个变量都对应一块地址空间，在变量被声明时，该地址空间被绑定到一个变量名。一个指针的值是另一个变量的地址，通过指针能够读或改变量的值而无需知道变量的名称。 指针的零值为nil。
```Go
x := 1
p := &x		//（p指向x/p保存了x的地址）type of p: *int
fmt.Println(*p)	// 1
*p = 2		// eq x = 2 (*p为x的别名)
fmt.Println(x)	// 2
```

### 生命周期
对于包级别的变量，其声明周期为整个程序的执行时间；而对于局部变量，其有一个动态的声明周期，在其声明时被创建，在其**不能被引用**时它占用的存储空间被回收。
变量的声明周期是通过它是否可达来确定的，因此局部变量可能在其循环结束之后存活或函数结束后存活（逃逸）。
```Go
var global *int
func f() {	// 函数结束后仍能够通过*global访问x（逃逸）
	var x int
	x = 1
	global = &x
}
func g() {	// 函数结束之后y变得不可访问
	y := new(int)
	*y = 1
}
```
每一次变量逃逸都需要一次额外的内存分配过程。编译器可以选择使用堆或栈上的空间来分配给变量，但是逃逸的变量一定在堆中，在编译时可以使用`-gcflags=-m`选项来查看变量逃逸情况。

### 赋值
赋值语句能够更新变量的值。
```Go
x = 1                       // 命名变量的赋值
*p = true                   // 通过指针间接赋值
person.name = "bob"         // 结构体字段赋值
count[x] = count[x] * scale // 数组、slice或map的元素赋值
// =可以与二元运算符结合
count[x] *= scale	// count[x] = count[x] * scale
// 自增、自减
// 自增和自减是表达式而不是语句，只能单独使用
v := 1
v++    // 等价方式 v = v + 1；v 变成 2
v--    // 等价方式 v = v - 1；v 变成 1
x := v++	// ！！！编译错误
```
Go能通过元组赋值同时更新多个变量的值：
```Go
i, j, k = 2, 3, 5
f, err = os.Open("foo.txt") // function call returns two values
_, err = io.Copy(dst, src)  // 能够使用_丢弃不需要的值	
```

### 类型
类型或表达式定义了对应存储值的属性特征（数值在内存的大小，在内部是如何表达，是否支持一些操作符以及它们自己关联的方法集等）。
一些变量可能有着相同的内部结构，但表示完全不同的概念（int可以表示循环的迭代索引、月份、文件描述符等）。一个类型声明语句创建了一个新的类型名称，和现有类型有相同的底层结构。虽然新类型和现有类型有相同的底层结构，但它们是不兼容的（不能相互比较或混在一个表达式运算，需要先使用`T()`函数进行类型转换，对于指针可能需要在T两边加小括号）。
```
type 类型名称 底层类型
```
通常在包级声明类型。
```Go
// Package tempconv performs Celsius and Fahrenheit temperature computations.
package tempconv

import "fmt"

type Celsius float64    // 摄氏温度
type Fahrenheit float64 // 华氏温度

const (
    AbsoluteZeroC Celsius = -273.15 // 绝对零度
    FreezingC     Celsius = 0       // 结冰点温度
    BoilingC      Celsius = 100     // 沸水温度
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```
可以为新类型定义新的行为，这些行为表示为一组关联到该类型的函数集合，称为该类型的方法集。
```Go
// Celsius类型的参数c出现在了函数名的前面
// 表示声明的是Celsius类型的一个名叫String的方法
// 该方法返回该类型对象c带着°C温度单位的字符串
func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }
```

### 包和文件
Go的包和其他语言的库或模块的概念类似，目的都是为了支持模块化、封装和代码重用。一个包的源代码保存在一个或多个以.go为后缀名的源文件中。
每个源文件都是以包的声明语句开始，用来指明包的名字，一个文件声明的类型和常量，在同一个包的其他源文件内也可以直接访问（包外需大写字母开头）。

### 变量基本类型
Go支持的基本变量类型有以下几种：
```Go
bool

string

int  int8  int16  int32  int64          // int和uint的大小取决于计算机字长(32/64)
uint uint8 uint16 uint32 uint64 uintptr

byte    // alias for uint8

rune    // alias for int32
        // represents a Unicode code point

float32 float64

complex64 complex128
```
除了特别需求（高性能、特别大的整数）以外，推荐使用的数据类型（为了代码简洁、易于阅读方便）：
- bool
- string
- int
- uint32
- byte
- rune
- float64
- complex128

### String
Go的字符串是不可以改变（strings.builder类型是可以改变的）的，但是可以重新赋值，使用`+`对两个字符串进行连接并生成一个新的字符串，或使用`fmt.Sprintf()`函数与其他类型变量进行拼接[Firmating Verbs](https://pkg.go.dev/fmt)。
```Go
s := "Hello world!!"
s[1] = 65   // !!!!ERROR
s += "abca" // "Hello world!!abca"
s = fmt.Sprintf("abc %s", s)    // "abc Hello world!"
```
### 变量类型转换
Go能够使用类型对应的函数对变量的类型进行转换。
```Go
a := 123
var float64 b = float64(a)
```
### 常量（constants）
常量是初始化后不会被修改的量，常量使用const关键词声明，常量无法使用短变量声明。
```Go
const pi = 3.14159
```
常量只能是基本类型（string，int等）
