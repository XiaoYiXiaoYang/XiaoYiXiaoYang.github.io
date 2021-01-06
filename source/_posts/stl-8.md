---
title: STL-栈、队列、堆、单链表
date: 2019-12-30 20:33:26
tags: [STL]
categories: [学习笔记]
---

 栈、队列、堆、单链表、带权值的队列基本介绍

<!--more-->

## stack

stack是一种先进后出的数据结构，stack允许新增、移除、取得最顶端元素，没有任何其他方法存取stack的其他元素，所以，**stack不允许有遍历行为**

deque是双向开口的数据结构，若以deque为底部结构并封闭其头端开口，便轻而易举的形成了一个stack

SGI STL以deque作为缺省情况下的stack的底部结构

stack以底部容器完成其所有工作，这种叫配接器，因此STL stack往往不被归类为容器，而被归类为容器配接器

|         |
| ------- |
| empty() |
| size()  |
| top()   |
| push()  |
| pop()   |
| ==      |
| <       |


stack不提供遍历功能，也没有迭代器

以list作为stack的底层容器: list也是双向开口的数据结构，以list为底部结构并封闭其头部端口，一样能够轻易形成一个stack

## queue

queue是一种先进先出的数据结构，queue允许新增、移除、取得最顶端元素、从最底端加入元素，没有任何其他方法存取queue的其他元素，**queue不允许有遍历行为**

deque是双向开口的数据结构，若以deque为底部结构并封闭其头端开口，便轻而易举的形成了一个queue

SGI STL以deque作为缺省情况下的queue的底部结构

queue以底部容器完成其所有工作，这种叫配接器，因此STL queue往往不被归类为容器，而被归类为容器配接器


|         |
| ------- |
| empty() |
| size()  |
| front() |
| back()  |
| push()  |
| pop()   |
| ==      |
| <       |


queue不提供遍历功能，也没有迭代器

以list作为queue的底层容器: list也是双向开口的数据结构，以list为底部结构并封闭其头部端口，一样能够轻易形成一个queue


## heap

heap并不属于STL容器组件，它是priority_queue的助手

堆是一个完全二叉树，每个结点的值都小于或等于其左右孩子结点的值

一个vector和一组heap算法是最好的实现方式

**push_heap()算法**

新加入的元素插入最后作为叶子结点

执行上溯程序：从最后一个分支结点开始，将该节点与左右孩子结点比较，若不满足堆的条件，则将根节点与左右孩子结点较大值交换


**pop_heap()算法**

将根节点与最后一个叶子结点对换

执行下溯程序：将根节点与左右孩子结点比较，若不满足堆的条件，则将该结点下放，直至下放到叶子结点为止

再对它执行一次上溯便大功告成


**sort_heap()算法**

sort_heap()就是不断的执行pop_heap()


**make_heap()算法**

建堆的过程

heap不提供遍历功能，也没有迭代器


## priority_queue

带权值的queue

缺省情况下使用一个max-heap完成，底部以vector为底部容器

priority_queue称为配接器，被归类为容器配接器

|         |
| ------- |
| empty() |
| size()  |
| top()   |
| push()  |
| pop()   |


## slist

单向链表

为了效率，slist不提供push_back(),只提供push_front()


slist结点和其迭代器设计，架构上比list要复杂

迭代器只有++ 

没有--



2019-10-14 15:59:37 星期一