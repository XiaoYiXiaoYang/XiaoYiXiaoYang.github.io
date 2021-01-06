---
title: STL--RB-tree/map/set
date: 2019-12-30 20:34:38
tags: [STL]
categories: [学习笔记]
---

 map/set低层都是红黑树

<!--more-->



## RB-tree

1.每个结点不是红色就是黑色

2.根节点为黑色

3.如果结点为红，其子节点必须为黑

4.任一结点至叶子节点的任何丼，所含黑结点个数必须相同


迭代器  left right  parent

header 和 根节点 互相指向

header的left指向最小结点，right指向最大结点

RB-tree一开始即要求用户必须明确设定所谓的KeyOfValue仿函数

因此，从实值中取出键值是毫无问题的

红黑树也像平衡二叉树一样


insert_equal() 允许键值重复

insert_unique()  不允许键值重复



## Map

根据键值自动排序，map所有元素都是pair

pair的第一元素被视为键值，第二元素视为实值

```
struct pair
{
	T1 first;
	T2 second;
}
```

map不允许两个元素拥有相同的键值

通过map迭代器改变键值不允许，改变实值可以

缺省情况下递增排序，以RB-tree为底层

使用其成员函数find()更有效率

下标操作符，可能是左值引用，也可能是右值引用..?



## Set

根据键值自动排序，键值即实值，实值即键值

算法：交集、联集、对称差集

缺省情况下递增排序，以RB-tree为底层

使用其成员函数find更有效率

不能通过迭代器改变set的元素值，元素值即键值，会破坏set组织



|               |
| ------------- |
| begin()       |
| end()         |
| rbegin()      |
| rend()        |
| empty()       |
| size()        |
| max_size()    |
| swap()        |
| insert()      |
| erase()       |
| clear()       |
| find()        |
| count()       |
| lower_bound() |
| upper_bound() |


## multimap

允许键值重复

使用RB-tree的insert_equal()

## multiset

允许键值重复

使用RB-tree的insert_equal()






2019-10-14 16:25:18 星期一