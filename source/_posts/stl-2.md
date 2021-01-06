---
title: STL--内存基本处理工具
date: 2019-12-30 20:25:53
tags: [STL]
categories: [学习笔记]
---



uninitialized_copy()，uninitialized_fill(),uninitialized_fill_n

<!--more-->

STL定义有五个全局函数，作用于未初始化空间上，这样的功能对于容器的实现很有帮助。前两个函数是用于构造的construct()和用于析构的destroy()，另三个函数是uninitialized_copy()，uninitialized_fill(),uninitialized_fill_n,分别对应于高层次函数copy()、fill()、fill_n()——这些都是STL算法。

**uninitialized_copy**


```
ForwardIterator uninitialized_copy(InputIterator first,InputIterator last,ForwardIterator result);
```

uninitialized_copy()使我们能够将内存的配置和对象的构造行为分离开来，

如果作为输出目的地的[result,result+(last-first))范围内的每一个迭代器都指向为初始化区域，

则uninitialized_copy()会使用copy constructor，给身为输入来源之[first,last)范围内的每一个对象产生一份复制品，放进输出范围中。



```
换句话说，针对输入范围内的每一个迭代器i，该函数会调用construct(&*(result+(i-first)),*i)，产生*i的复制品，放置于输出范围的相对位置上。
```

如果你需要实现一个容器，uninitialized_copy()这样的函数会为你带来很大的帮助，因为容器的全区间构造函数通常以两个步骤完成：

1.配置内存块，足以包含范围内的所有元素

2.使用uninitialized_copy()，在该内存区块上构造元素。

C++标志规格书要求uninitialized_copy()具有“commit or rollback”语意，意思是要么“构造出所有必要的元素”，要么（当有任何一个copy constructor失败时）“不构造任何东西。




**uninitialized_fill**

```
ForwardIterator uninitialized_fill(ForwardIterator first,ForwardIterator last,const T& x);
```

uninitialized_fill()也能够使我们将内存配置与对象的构造行为分离开来

如果[first,last)范围内的每个迭代器都指向未初始化的内存，那么uninitialized_fill()会在该范围内产生x（上式第三个参数）的复制品。


```
换句话说，uninitialized_fill()会针对操作范围内的每个迭代器i，调用construct(&*i,x)，在i所指之处产生x的复制品。
```

与uninitialized_copy()一样，uninitialized_fill()必须具备“commit or rollback”语意，

换句话说，它要么产生出所有必要元素，要么不产生任何元素，如果有任何一个copy constructor丢出异常（exception），uninitialized_fill()，必须能够将已产生的所有元素析构掉。


```
针对 char * 和 wchar_t *型别 可以采用最有效率的做法 memmove
```








**uninitialized_fill_n**

```
ForwardIterator uninitialized_fill_n(ForwardIterator first,Size n,const T& x);
```

uninitialized_fill_n()会调用 copy constructor，在该范围内产生x（上式第三个参数——的复制品。

```
也就是说，面对[first,first+n)范围内的每个迭代器i，uninitialized_fill_n()会调用construct(&*i,x),在对应位置产生x的复制品。
```

commit or rollback”语意：要么产生所有必要的元素，否则就不产生任何元素











2019-10-24 15:01:39 星期四