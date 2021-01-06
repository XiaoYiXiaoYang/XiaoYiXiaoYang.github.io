---
title: C++11（基础）
date: 2019-12-14 22:08:12
tags: [C++]
categories: [学习笔记]
---

 阅读学习C++ Primer，顺便学习C++11的新特性

<!--more-->

- long long 类型

long long是长整型，最小尺寸64位，也就是8个字节。

- 列表初始化

```
int s=0;
int s={0};
int s{0};
int s(0);
```
以上定义了一个变量s，并将其初始化为0，其中用花括号来初始化变量是c++11的新特性，这种初始化形式叫列表初始化。
现在，初始化对象或者为某些对象赋新值时，都可以使用这样一组由花括号括起来的初始值。

- nullptr常量

nullptr是一种特殊类型的字面值，它可以被转换成任意其他的指针类型。
得到空指针最直接的方法就是用字面值nullptr来初始化指针

过去的程序还会用到一个名为NULL的预处理变量来给指针赋值，这个变量在头文件cstdlib中，它的值就是0

在用到一个预处理变量时，预处理器会自动的将它替换为实际值，因此用NULL初始化指针和用0初始化指针是一样的

注意：
把int变量直接赋值给指针是错误的，即使int变量的值为0


- constexpr变量

常量表达式是指值不会改变，且在编译过程中就能得到计算结果的表达式

字面值属于常量表达式，用常量表达式初始化的const对象也是常量表达式

```
const int limit=10;   //常量表达式
const int ll=limit+1;  //常量表达式
int ss =1;    //不是常量表达式
const int sz = get_size();  //sz不是常量表达式，尽管他本身是一个常量，但是它的值在运行时才能获取到
```

允许将变量声明为constexpr类型以便由编译器来验证变量的值是否是一个常量表达式

尽管指针和引用可以声明为constexpr类型，但是指针的初始值必须是nullptr或者0，或者是存储于某个固定地址的对象

一般函数体内的变量并非存放在固定地址，constexpr指针不能指向这样的变量

定义与函数体外的对象其地址固定不变，可以用来初始化constexpr指针


**指针和constexpr**


限定符仅对指针有效，有指针所指的对象无关

```
const int *p=nullptr;   //指向整型常量的常量指针
constexpr int *q =nullptr;   //指向整数的常量指针
```

- auto

**更一般的，顶层const可以表示任意的对象是常量**

用它就能让编译器替我们去分析表达式所属的类型

auto让编译器通过初始值来推算变量的类型

auto一个引用会得到引用绑定对象的类型


auto一般会忽略顶层const，保留底层const

```
const int ci=i; &cr=ci;
auto b = ci;  //b是一个整数（顶层const被忽略）
auto c = cr;   //c是一个整数（cr是ci的别名，ci本身是一个顶层const）
auto d =&i;   //d是一个整型指针，
auto e = &ci;   //e是一个指向整数常量的指针
```

- decltype

选择并返回操作数的数据类型

如果decltype使用的表达式是一个变量，则decltype返回该变量的类型（包括顶层const和引用在内）

```
const int ci =0;&cj=ci; 
decltype(ci) x = 0;  //x的类型是const int
decltype(cj) y = 0;  //y的类型是const int &,y绑定到变量x
decltype(cj) z;      //错误，z是一个引用必须初始化
```

**decltype和引用**

有些表达式向decltype返回一个引用类型，一般当这种情况发生时，意味着该表达式的结果对象能作为一条赋值语句的左值

```
int i=42, *p =&i,&r=i;
decltype(r+0) b;  //正确，加法的结果是int，因此b是一个（未初始化的）int

decltype(*p) c; //错误，c是int &，  如果表达式的内容是解引用操作，则decltype将得到引用类型
```

如果decltype使用的是一个不加括号的变量，则得到的结果就是该变量的类型，如果给变量加上了一层或多层括号，编译器会把它当成是一个表达式，变量是一种可以作为赋值语句左值的特殊表达式，**这样的decltype得到的是引用类型**

```
decltype ((i)) d; //错误，d是int &，必须初始化
decltype (i) e;   //正确，e是一个未初始化的int
```





