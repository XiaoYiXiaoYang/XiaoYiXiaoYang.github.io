---
title: C++Primer（第12章 动态内存）
date: 2019-12-15 15:42:40
tags: [C++ Primer]
categories: [学习笔记]
---

程序用堆来进行内存动态分配

<!--more-->

#### 动态内存与智能指针

new运算符，咋动态内存中为对象分配空间并返回一个指向该对象的指针

delete运算符接收一个动态对象的指针，销毁该对象，并释放与之关联的内存



shared_ptr允许多个指针指向同一对象

unique_ptr允许独占指向的对象

weak_ptr是一种弱引用，指向shared_ptr所管理的对象



```
shared_ptr<int> p1;
shared_ptr<list> p2;
```



**make_shared函数**

最安全的分配和使用动态内存的方法是调用一个名为make_shared的标准库函数

此函数在动态内存中分配一个对象并初始化它，返回此对象的shared_ptr

```
shared_ptr<int> p1 = make_shared<int>(42)
```



**shared_ptr拷贝和赋值**

每个shared_ptr都会记录有多少个其他shared_ptr指向相同的对象

拷贝一个shared_ptr、用一个shared_ptr初始化另一个shared_ptr，将它作为参数传递给另外一个函数，作为函数的返回值，它所**关联的引用计数都会递增**

当引用计数为0时，它就会释放自己管理的对象【析构函数、释放内存】



##### 直接管理内存;

new

delete



**使用new动态分配和初始化对象**

可以对动态分配的对象进行直接初始化 int *p = new int(5);

也可以对其值初始化  string a = new string();



**动态分配的const对象**

动态分配的const对象必须进行初始化



**内存耗尽**

内存耗尽，new失败的时候回抛出一个bad_alloc异常

可以改变使用new的方式来阻止它抛出异常

int *p =  new (nothrow) int;



**野指针**

delete之后指针值就无效了，需要将其手动置为空

如果多个指针指向相同的内存，置空只能对这个指针有效，对其他仍然指向内存的指针无效



**shared_ptr和new**

不能将一个内置类型的指针赋给一个智能指针，智能使用直接初始化

```
shared_ptr<int> p1 = new int(1024);
```



**不要混合使用普通指针和智能指针**



```
void process(shared_ptr<int> ptr)
{
	
}
```

函数参数是值传递的，因此实参会被拷贝到ptr中。拷贝一个shared_ptr会递增其引用计数，在process运行中，引用计数至少为2，process结束时，ptr的引用计数会递减，但不会变为0，因此，当局部变量ptr销毁时，ptr指向的内存不会被释放

正确的方法是传递给该函数一个shared_ptr

```
shared_ptr<int> p(new int(42));
process(p);
int i = *p; //正确
```



错误的方法是传递给该函数一个内置类型的指针

```
int *x(new int(42));
process(p);  //错误，不能将内置类型指针赋给智能指针
proces(shared_ptr<int>(x));  //正确，但是不会释放内存
int j = *x;  //错误，x是一个空悬指针
```



**智能指针和异常**

- 如果使用智能指针，即使程序抛出异常，智能指针类也能确保在内存不再需要的时候将其释放
- 当发生异常时，我们直接管理的内存不会自动释放的，如果使用内置指针管理内存，且在new之后，delete之前发生异常，则内存不会被释放



**unique_ptr**

某个时刻只能有一个unique_ptr指向一个给定对象

不能拷贝和赋值unique_ptr



reset()方法释放智能指针指向的对象

可以将指针的所有权从一个unique_ptr转移给另一个unique_ptr

```
unique_ptr<string> p2(p1.release);  //release将p1置空
```



不能拷贝unique_ptr的规则有一个例外，可以拷贝或赋值一个将要被销毁的unique_ptr

```
unique_ptr<int> clone(int p)
{
	return unique_ptr<int>(new int(p));
}
```



**weak_ptr**

weak_ptr指向一个shared_ptr指向的对象

解决shared_ptr相互引用时候的死锁



#### 动态数组

标准库allocator类

```
int *p = new int[10];
delete []p;
```



**allocator类**

```
allocator<string>alloc;
alloc.allocate(10);   //分配n个未初始化的string
```



**allocator分配未构造的内存**

allocator分配的内存时未构造的

在新标准库中，allocator接受一个指针和另个或多个额外参数，在给定位置构造一个元素。额外参数用来初始化构造的对象

```
alloc.construct(q,10,'c');   
```







