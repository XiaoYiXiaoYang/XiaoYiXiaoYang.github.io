---
title: C++对象模型(Data语意学)
date: 2019-12-14 10:49:33
tags: [C++对象模型]
categories: [学习笔记]
---

成员布局的大小

1.语言本身造成的负担（虚基类指针设置）

2.编译器对于特殊情况的优化处理

3.对齐限制

<!--more-->



```c++
class X{};
class Y:public virtual X{};
class Z:public virtual X{};
class A:public y,public z{};
```

求sizeof结果

sizeof X结果 1

sizeof Y结果 8

sizeof Z结果 8

sizeof A结果 12

X类的成员布局

| 1字节（空类的this指针） |
| :---------------------: |
|                         |



Y类的成员布局



|   4字节虚函数表指针    |
| :--------------------: |
| 1字节的父类（X类）指针 |
|       3字节对齐        |



Z类的成员布局

|   4字节虚函数表指针    |
| :--------------------: |
| 1字节的父类（X类）指针 |
|       3字节对齐        |



A类的成员布局

|        1字节爷爷类指针（X类）         |
| :-----------------------------------: |
| 4字节Y类大小，减去因虚基类X配置的大小 |
| 4字节Z类大小，减去因虚基类Y配置的大小 |
|            A类自己的大小0             |
|          A类对齐的大小3字节           |



X类为空的虚基类

这是C++面向对象设计的一个风格，它提供了虚拟接口，没有定义任何数据



某些新近的编译器对此情况特殊处理（如Visual C++），一个空的虚基类被看做派生类的最开头的部分，也就是它不花费额外的空间，也就省去了上述Y类，Z类开头所占用的1字节



------



#### 数据成员的绑定



```c++
extern float x;  //声明x
class Point
{
public:
    Point(float);
    float X(){return x};
private:
    float x;  //又一个x
};
```



函数返回的当然是class内部的x，但是编译器进化过程中不是这样的

有过两种防御性编程风格

1.

```c++
class Point
{
    float x;	//将class成员声明放在开头
 public:
};
```



2.

```c++
class Point
{
public:
	float X();   //将函数定义放在clas声明外
private:
    float x;
};

float Point::X
{
    return x;
}
```



后来演练C++标准，在C++声明完全看见之前，不会评估值，

也就是将类内定义的函数不分析，在类声明结束之后才开始分析

但是这对于**参数列表**不为真，类內声明的函数的参数列表还是会被分析



#### 数据成员的布局



规则：较晚出现的成员在类对象中有较高的地址



vptr一般被放在类最后，不过也有一些编译器把vptr放在类开始



#### 数据成员的存取



```c++
Point origin;
Point *pt = &origin;

origin.x = 0.0;
pt->x = 0.0;
```



**1.static data member**

每一个static data member只有一个实例，存在于程序的数据段之中

每一个static data member的存取都是一样的效率



如果有两个类，声明了一样的static member，那么当它们都放在全局静态存储区就会导致命名冲突，解决办法是编译器暗中对每一个static member重命名



**2.nonstatic data member**



想要对一个nonstatic data member 进行存取操作，编译器需把类对象的起始地址加上数据成员的偏移位置(offset)，如果：

origin.y = 0.0;

则地址 &origin.y为

&origin + (&Point::_y -1)

*其中-1操作时因为，指向data member的指针，其offset值总是被加上1*



#### 继承与data member



```c++
class Point2d
{
	float x,y;
};

class Point3x
{
	float z;
};
```



**单一继承且不含虚函数**

布局缺陷

如果

```
class A
{
	int val;
	char a;
	char b;
	char c;
};
```

分裂成继承结构

```
class A
{
	int val;
};
class B
{
	char a;
};
...
```



则由于对齐可能产生D类对象的成员布局

|  char c,1字节  |
| :------------: |
|   3字节对齐    |
|  char b,1字节  |
|   3字节对齐    |
| char a，1字节  |
|   3字节对齐    |
| int val，4字节 |



这些对齐是有必要的，子类和基类指针之间操作需要这样的行为

假如没有这样的布局，将一个派生类指针赋给基类指针，就会出错，



这样的存取不会增加空间或者时间上的额外负担



**单一继承含虚函数**



含虚函数会增加空间和时间上的负担



- 导入和Point2d有关的虚函数表，用它存放每一个虚函数的地址
- 在每一个类对象中导入一个vptr，提供执行期的链接
- 加强构造函数，使得正确设定vptr值
- 加强析构函数，使得消除vptr值



**vptr放在尾端**

可以保留基类对象布局



**vptr放在开头**

可以省去偏移计算



**多重继承**

多重继承的复杂度在于派生类和第二或后继基类对象之间的转换

```c++
class Point2d;
class Point3d:public Point2d
class Vertex;
class Vertex3d:public Point3d,public Vertex;
```



Vertex3d类对象和Point3d类对象之间的转换和单一继承时相同，两者都是相同的起始地址

而Vertex3d类对象和Vertex类对象之间的转换需要地址指定，加上或者减去介于中间的base class subobjects的大小

```
Vertex3d v3d;
Vertex *px;

px = &v3d;
px = (Vertex *)((char *)&v3d) + sizeof(Point3d));
```



**虚拟继承**

虚拟继承会有共同的基类，一种布局策略是先安排好派生类的不变部分，再建立其共享部分

cfront编译器存取共享部分的策略是在派生类对象中安插一些指针，每个指针指向一个虚基类

但是缺点是

1.每个对象必须为其每一个虚基类背负一个额外的指针

2.虚拟继承串链的加长，导致间接存取层次的增加



解决办法：

1.安插一个指针指向虚拟基类表

2.在虚函数表中放置虚基类的偏移

如果是正值，索引到虚函数，如果是负值，索引到虚基类偏移



#### 对象成员的效率

聚合、封装、继承



一般继承不会影响效率

虚拟继承对效率耗费多



#### 指向data member的指针



为了区分“**没有指向任何数据成员的**”指针 和一个**“指向第一个数据成员的的指针**”

每一个真正的member offset都被加上1



#### 指向member的指针的效率问题



每一层虚拟继承都会导入一个额外的间接性



















