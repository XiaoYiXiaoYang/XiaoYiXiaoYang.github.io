---
title: Go语言基础
date: 2021-01-05 22:57:36
tags: [GoLang]
categories: [学习笔记]
---

<center>
Go语言介绍、基础语法
</center>

<!--more-->



## Go语言介绍


![img](go-1/Tue,%2005%20Jan%202021%20230620.jpeg)

- Google开源
- 编译型语言

特点：

- 语法简洁
- 开发效率高
- 执行性能好


学习源：www.liwenzhou.com


## Go开发环境搭建

Go开发包镜像地址：https://golang.google.cn/dl/

安装好后，cmd下运行`go version`显示版本号


## go语法


### 包、变量、函数

```
推荐import package方式

import (
    "fmt"
    "math"
)
```

**函数**

func 函数名(参数名 类型,...) 返回值... {
}
 
 
//1.多个参数同一类型时，支持简写(x, y int)
//2.支持多个返回值时 声明为func swap(x, y string) (string, string)   返回为 return x, y
//3.支持return named return values   声明为func test(sum int)(x, y int)   返回为 return


**变量**

var 变量名... 类型   // 声明
var x, y int = 1, 2 //声明加初始化
x := 1              //短变量声明 类型推导
 
 
**基本类型**
```
bool
 
string
 
int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr
 
byte // alias for uint8
 
rune // alias for int32
     // represents a Unicode code point
 
float32 float64
 
complex64 complex128
 
 
 
 
zero values：
0 for numeric types,
false for the boolean type, and
"" (the empty string) for strings.
nil for other type
``` 
 
类型转换：
T(v)


**常量**

const 变量 = 值
 
 
//1.不能使用短变量声明法


### 流程控制

**for**

for init statment; condition statement; post statement {
}
 
 
//1.初始和结束执行的语句可以省略(保留分号)
//2.只有一句condition，像使用while一样(没有分号)
//3.连condition语句都没有(死循环)


**if**

if condition {
}
//1.没有小括号，但是{}是必须的
//2.变形 if init statement; condition stament {}(声明的短变量作用域在if else块内)


**switch**

switch init statement;condition{
    case 值1：
    case 值2
}
//1.go自动在每个case后break
//2.go的条件不必是const的，也不一定是integer
//3.switch语句也可以带短变量声明语句，也可以省略condition(等于switch true)


**defer**
```
func test() {
    fmt.Println("123")
}
 
func main() {
    defer fmt.Println("world")
 
    fmt.Println("hello")
    test()
}
``` 
 
//1.defer语句会立即求值，但是推迟到函数返回之前执行。
//2.多个defer语句会压栈，直到函数返回时，多个defer会按出栈顺序调用


### 复合数据类型

**pointer**
i= 42
p := &i
 
 
//1.默认未初始化为零值


**struct**
```
type Vertex struct {
    X int
    Y int
}
 
func main() {
    v := Vertex{1, 2}
    p := &v
    p.X = 666
    fmt.Println(v)
}
```
//1.可以使用v.X 也可以使用指针p.X访问成员


**array**
```
var a [2]string
a[0] = "Hello"
a[1] = "World"
fmt.Println(a)
```
 
//1.array是值 数组长度固定


**slice**
```
primes := [6]int{2, 3, 5, 7, 11, 13}
var s []int = primes[:4]  //切片
fmt.Println(s)
``` 
 
//1.切片是引用，切片底层指向数组，长度可扩容
//2.切片a[low:high];省略low为0，省略high为切片的len
//3.make 构造，make([]int, len, cap)
//4.切片元素为切片
//5.append 可以添加一个元素、多个元素、一个切片。当切片容量不足，则新创建一个大数组，切片指向新数组


**range**
//1.range 数组；slice；返回 inedx, value
//2.range map；返回 key, value
//3.匿名变量 _ ，接收不使用的值


**map**
//1.删除元素  delete(map1, key)
//2.测试key值是否存在 value， ok = map1[key]


**function**
func 函数名(参数) 返回值 {}
 
 
//1.函数可以作为参数，也可以作为返回值
//2.函数闭包：闭包是一个函数值，它从其主体外部引用变量。该函数可以访问并分配给引用的变量。函数是绑定到变量的


### 方法和接口
**method**
func (receiver) 函数名(参数) 返回值 {}
 
 
//1.作用于特定的receiver的函数
//2.特定的receiver可以不是struct，可以是基本数据类型。(是否需要使用type重命名一下)
//3.指针receiver 和 值receiver；指针receiver可以修改值，(传参无需关心传的是指针还是值)(函数则不行，函数必须要确定传参类型匹配)


**interface**

```
type T struct {
    S string
}
 
 
type add interface {
    Abs() float64
}
 
 
func (t T) Abs() { // 类型通过实现其method来实现接口
    fmt.Println(t.S)
}
 
 
var i I = T{"hello"}
i.M()
``` 
 
//1.接口值可以视为(value, type)，在接口值上调用方法会在其基础类型上执行相同名称的方法。
//2.var i I；一个空的接口值，其value和type都是nil，调用会产生运行时错误
//3.var i interface{}；一个空接口，可以保存任何类型的值
//4. s, ok := i.(string)； 判断接口是否含有类型string
//5.fmt就是其他都实现了String接口
type Stringer interface {
    String() string
}
 
 
//6.go中的error是其他实现了Error接口
type error interface {
    Error() string
}




### 并发
**goroutines**


**channels**


**select**



