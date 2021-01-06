---
title: STL-------List
date: 2019-12-30 20:31:15
tags: [STL]
categories: [学习笔记]
---

 List是双向链表

<!--more-->

**List**

```
#include <list>
```

| 构造函数                                                     |      |
| ------------------------------------------------------------ | ---- |
| explicit list (const allocator_type& alloc = allocator_type()); |      |
| explicit list (size_type n, const value_type& val = value_type(),const allocator_type& alloc = allocator_type()); |      |
| template <class InputIterator> list (InputIterator first,InputIterator last,const allocator_type& alloc = allocator_type()); |      |
| list (const list& x);                                        |      |

**属性**

容量:没有了

|          |                                                              |
| -------- | ------------------------------------------------------------ |
| length() | 没有                                                         |
| size()   | 元素个数                                                     |
| resize() | 重新设置元素个数，设置的比现有的小，则输出截断，size变小；如果设置的比现有的大，则size变大 |

**输出**

|          |                |
| -------- | -------------- |
| 循环     | 迭代器         |
| for_each |                |
| []下标   | 不支持         |
| back()   | 返回尾巴的元素 |
| front()  | 返回头部的元素 |

**修改**

|          |              |
| -------- | ------------ |
| 头添加   | push_front() |
| 尾添加   | push_back()  |
| 中间添加 | insert       |

**删除**

|        |             |
| ------ | ----------- |
| 头删除 | pop_front() |
| 尾删除 | pop_back()  |
|        | erase()     |
|        | clear()     |
|        | remove()    |
|        | unique()    |

```
list<int> lst;
	lst.push_back(0);
	lst.push_back(2);
	lst.push_back(2);
	lst.push_back(3);
	lst.push_back(2);
	lst.push_back(2);
	lst.push_back(0);
	lst.unique();

	for (list<int>::iterator it = lst.begin(); it != lst.end(); it++)
	{
		cout << *it << " ";
	}
	
//结果输出 0 2 3 2 0
```

**赋值**

|        |      |
| ------ | ---- |
| assign |      |
| =      |      |

**交换**

|      |                    |
| ---- | ------------------ |
| swap | 交换两个list的内容 |
|      |                    |

**其他**

|      |           |
| ---- | --------- |
| 合并 | merge()   |
| 拼接 | splice    |
|      | reverse() |
|      | sort()    |















2019-09-22 21:25:57 星期日