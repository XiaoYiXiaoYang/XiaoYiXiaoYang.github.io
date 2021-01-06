---
title: STL-------vector
date: 2019-12-30 20:29:55
tags: [STL]
categories: [学习笔记]
---

 尽可能使用vector

<!--more-->

**Vector**

```
头文件 #include<vector>
```

**构造函数**

```

1、vector( const Allocator& = Allocator() );
2、vector( size_type n,constT& value = T(), const Allocator& = Allocator() );
3、template <class InputIterator>
     vector ( InputIterator first, InputIterator last, const Allocator& = Allocator() );
4、vector ( const vector<T,Allocator>& x );
```

**容量**

|          |                                                              |
| -------- | ------------------------------------------------------------ |
| capacity |                                                              |
| 大小     | 在VS中测试，当空间不够时，会分配现有空间的一半加上现有空间，如果还不够，则分配足够大小；在<<STL源码剖析>>中，源码策略为分配现有空间的二倍，如果不够则分配旧长度加新增元素个数； |
| 策略     | 不论哪种分配方式，都是三步“重新配置，移动数据，释放原空间”   |
| 优点     |                                                              |


| Length   |                                                              |
| -------- | ------------------------------------------------------------ |
| length() | vector没有这个成员函数                                       |
| size()   | 元素个数                                                     |
| resize() | 重新设置容器大小。如果resize的小于现有大小，则容器的size变小，元素也被截断，capacity不变；如果resize的大于现有大小，则容器的size和capacity都会变大，变大的策略还是看增加现有空间一半，否则等于旧长度加新增长度；resize是改变容器的大小，且在创建对象，resize函数可以有两个参数，第一个参数是容器新的大小， 第二个参数是要加入容器中的新元素，如果这个参数被省略，那么就调用元素对象的默认构造函数 |
| reserve  | 设置预留空间。设置的比现有空间小，则size和capacity都不变，设置的大，则size不变，capacity增大为设置的大小，reserve是容器预留空间，但在空间内不真正创建元素对象 |

输出

|        |                                    |
| ------ | ---------------------------------- |
| <<     | 不能直接输出全部，只能一个一个输出 |
| []下标 | 访问单个元素                       |
| at()   | 访问单个元素                       |

修改

| insert                                                       |      |
| ------------------------------------------------------------ | ---- |
| iterator insert( iterator loc, const TYPE &val );            |      |
| void insert( iterator loc, size_type num, const TYPE &val ); |      |
| void insert( iterator loc, input_iterator start, input_iterator end ); |      |

**append**

| 没有 |      |
| ---- | ---- |
|      |      |
|      |      |

**赋值**

|        |                        |
| ------ | ---------------------- |
| =      | 同类型对象之间支持赋值 |
| >>     | 暂留                   |
| assign |                        |

查找

| find |      |
| ---- | ---- |
| 没有 |      |
|      |      |


**迭代器失效**


1.内存空间被重新分配的失效情况

如果添加的元素超过了容器的容量，则容器会新申请更大容量的内存空间，再将之前旧容器的元素拷贝进来，再添加新元素，此时，迭代器全部失效

2.内存空间未被重新分配

如果添加的元素没有超过元素容量，则直接添加到了现在容器的末尾，则其尾后迭代器会失效；如果是插入操作，插入位置后的迭代器都失效；如果是删除，删除位置后的迭代器都失效









2019-09-22 17:28:44 星期日