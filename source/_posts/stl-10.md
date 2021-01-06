---
title: STL--hashtable
date: 2019-12-30 20:35:37
tags: [STL]
categories: [学习笔记]
---

 hash表实现的map、set等

<!--more-->

## Hashtable

字典结构

冲突解决办法：

**线性探测法**  （h+1 ..）

**二次探测法**   （h+1^2）

**开链法**      （vector + 链表）

SGI STL以开链法完成hashtable

一个数组和很多个单链表，把每个单链表叫bucket，（桶）

**++**

1.从当前节点前进一个位置，链表的next

2.如果是末尾，则跳至下一个bucket桶


迭代器

```
node *next;  //自己实现的  不是list
hashtble *ht;  
```

hashtable 成员

```
vector<node *,Alloc> buckets;
size_type num_elements;
```


SGI以质数来设计表格大小，并先将28个质数计算好

**每个bucket的最大容量和buckets vector 的大小相同**


|             |
| ----------- |
| insert()    |
| resize()    |
| bkt_num()   |
| clear()     |
| copy_from() |
| find()      |
| count()     |


hash functions

无法处理string short double，想要处理这些，用户需要自己实现hash function



## hash_set

以hashtable为底层

|          |           |
| -------- | --------- |
| set      | RB-tree   |
| hash_set | hashtable |


## hash_set

以hashtable为底层

|          |           |
| -------- | --------- |
| map      | RB-tree   |
| hash_map | hashtable |


## hash_multiset

底层hashtable，不会自动排序

insert()使用hashtable 的insert_equal()



## hash_multimap


底层hashtable，不会自动排序

insert()使用hashtable 的insert_equal()




2019-10-14 16:39:00 星期一