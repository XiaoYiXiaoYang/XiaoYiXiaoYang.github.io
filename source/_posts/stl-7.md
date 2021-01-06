---
title: STL--deque
date: 2019-12-30 20:32:07
tags: [STL]
categories: [学习笔记]
---

 deque是双向开口的数据结构，分段的连续线性空间

<!--more-->

头文件

```
#include<deque>
```

构造函数

```
explicit deque (const allocator_type& alloc = allocator_type());

explicit deque (size_type n);
         deque (size_type n, const value_type& val,
                const allocator_type& alloc = allocator_type());

template <class InputIterator>
  deque (InputIterator first, InputIterator last,
         const allocator_type& alloc = allocator_type());

deque (const deque& x);
deque (const deque& x, const allocator_type& alloc);

deque (deque&& x);
deque (deque&& x, const allocator_type& alloc);

deque (initializer_list<value_type> il,
       const allocator_type& alloc = allocator_type());
```


**容量**  没有了

**length()**  没有

**size()**	元素个数

**resize()**  重新设置容器大小。

**reserve()**  没有了【deque它动态的以分段连续空间组合而成，随时可以增加一段新的空间并链接起来】

输出

| <<     | 不能直接输出全部，只能一个一个输出 |
| ------ | ---------------------------------- |
| []下标 | 访问单个元素                       |
| at()   | 访问单个元素                       |

增

**push_back()**  
**push_front()**  

删除

**pop_front()**
**pop_back()**

提供头插、尾插   和头删  尾删

deque的迭代器

| cur  | first | last | node |
| ---- | ----- | ---- | ---- |
|      |       |      |      |


2019-10-14 16:05:19 星期一