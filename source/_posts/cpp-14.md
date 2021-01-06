---
title: struct在C和C++中的不同
date: 2019-12-14 22:50:04
tags: [C++]
categories: [技术分享]
---

 虽然关键字不变，但是含义不同

<!--more-->

### C中的struct

c中struct是自定义数据类型   UDT

C中struct是没有权限设置的

C中struct中只能是一些变量的集合体，可以封装数据，但是不能隐藏数据，成员也不能函数

一个结构标记声明后，在C中必须在结构标记前加上struct，才能做结构类型名（除：typedef struct class{};）;

### C++中的struct

C++中struct是抽象数据类型 ADT   (C++把struct当成类来处理)

C++中struct默认访问符为public，C++中struct增加了访问权限

C++中struct可以有成员函数

C++中结构体标记（结构体名）可以直接作为结构体类型名使用


### 代码


**C**

在C中定义一个变量要用typedef

```
typedef struct Student {
	int a;
} Stu;
```

于是在声明变量的时候就可：Stu stu1;
(如果没有typedef就必须用struct Student stu1;来声明)
这里的Stu实际上就是struct Student的别名。Stu==struct Student

另外这里也可以不写Student（于是也不能struct Student stu1 了，必须是Stu stu1，如下）

```
typedef struct {
	int a;
} Stu;
```

如果我们写

```
typedef struct  
{
　　int num;
　　int age;
}aaa,bbb,ccc;
```

相当于

```
typedef struct  
{
　　int num;
　　int age;
}aaa；
typedef aaa bbb;
typedef aaa ccc;
```

也就是说aaa,bbb,ccc三者都是结构体类型。声明变量时用任何一个都可以,在c++中也是如此


typedef struct和struct的区别：

```
typedef struct tagMyStruct
{ 
　　　int iNum;
　　　long lLength;
} MyStruct;
```

上面的tagMyStruct是标识符，MyStruct是变量类型（相当于（int,char等））。


```
在C中，这个申明后申请结构变量的方法有两种：
（1）struct tagMyStruct 变量名
（2）MyStruct 变量名
在c++中可以有
（1）struct tagMyStruct 变量名
（2）MyStruct 变量名
（3）tagMyStruct 变量名
```


**C++**


```
struct Student {
	int a;
};
```

于是就定义了结构体类型Student，声明变量时直接Student stu2；

```
struct Student {   
　　int a;   
}stu1;//stu1是一个变量  注意！！！！！！！！！
 
typedef struct Student2 {   
　　int a;   
}stu2;//stu2是一个结构体类型=struct Student2  
```

使用时可以直接访问stu1.a
但是stu2则必须先 stu2 s2;
然后 s2.a=10;




2019-07-29 23:01:31 星期一