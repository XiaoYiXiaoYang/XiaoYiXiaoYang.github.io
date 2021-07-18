---
title: QT---信号槽
date: 2020-01-08 11:46:13
tags: [Qt]
categories: [学习笔记]
---

<center>
用过Qt，十分友好。与MFC不同，QT是跨平台的的开发类库，还支持移动和嵌入式操作系统
</center>

<!--more-->

### 工程

新建项目

- Qt Widgets Application  支持桌面级平台的图形用户界面

- Qt Console Application  控制台应用程序

- Qt Quick Application 创建可部署的QT Quick2应用程序

- Qt Quick Controls Application 创建基于Qt Quick Controls 2 的 可部署的QT Quick2应用程序

- Qt Canvas 3D Application 创建QT Canvas 3D QML 项目

选择创建界面的基类

- QMainWindow 类是主窗口类，主窗口具有主菜单栏，工具栏和状态栏

-  QWidget 是所有具有可视界面类的基类

- QDialog 是对话框类，可建立一个基于对话框的界面


### 文件组成

工程名.pro文件   项目管理文件

Headers文件夹    头文件

	dialog.h文件

Sources文件夹   源文件

	dislog.cpp
	main.cpp

Forms文件夹  界面文件

	dialog.ui

### 界面UI设计

- 组件面板

- 待设计窗体

- Signals 和 Slots 编辑器与Action编辑器是位于待设计窗体下的两个编辑

- 对象浏览器    Object Inspector

- 属性编辑器  Property Editor


### 信号与槽机制

信号与槽（Signal and Slot）

信号就是在特定情况下被发射的事件

槽就是对信号响应的函数

信号与槽通过QObject::connect()函数实现的，connect是一个static函数

connect(发出信号的对象，发出的信号，接收信号的对象，接收信号后调用的函数)

（1）一个信号可以与另一个信号相连，代码如下：

connect(Object1,SIGNAL(signal1),Object2,SIGNAL(signal1));

表示Object1的信号1发送可以触发Object2的信号1发送。

（2）同一个信号可以与多个槽相连，代码如下：

connect(Object1,SIGNAL(signal2),Object2,SIGNAL(slot2));
connect(Object1,SIGNAL(signal2),Object3,SIGNAL(slot1));

(槽函数按照建立连接时的顺序依次执行)

（3）同一个槽可以响应多个信号，代码如下：

connect(Object1,SIGNAL(signal2),Object2,SIGNAL(slot2));
connect(Object3,SIGNAL(signal2),Object2,SIGNAL(slot2));

但是，常用的连接方式为：

connect(Object1,SIGNAL(signal),Object2,SLOT(slot));

其中，signal为对象Object1的信号，slot为对象Object2的槽。


- 严格情况下，信号与槽的参数个数和类型需要一致，至少信号的参数不能少于槽的参数，如果不匹配，会出现编译错误和运行错误



### 信号槽关联方式



- Qt:: DirectConnection (立即调用） 当信号发出后，相应的槽函数将立即被调用；

  emit语句后的代码将在所有槽函数执行完毕后被执行，信号与槽函数关系类似于函数调用，**同步执行**。

- Qt::QueuedConnection (异步调用） 当信号发出后，排队到信号队列中，需等到接收对象所属线程的事件循环取得控制权时才取得该信号，调用相应的槽函数。

  emit语句后的代码将在发出信号后立即被执行，无需等待槽函数执行完毕；

  此时信号被塞到信号队列里了，信号与槽函数关系类似于消息通信，**异步执行。**

- Qt::BlockingQueuedConnection (同步调用） 插槽作为队列连接调用，除非当前线程阻塞，直到插槽返回。

  使用此类型连接同一线程中的对象将导致死锁。

- Qt: :AutoConnection (默认连接） 如果信号的发送者和信号的接受者的对象同属一个线程，那个工作方式与直连方式相同；否则工作方式与排队方式相同。

  

- Qt:: UniqueConnection (单一连接） 该行为与自动连接相同，但是只有在不复制现有连接的情况下才会进行连接。

  

  请注意，如果事件循环在接收方的线程中运行，那么当发送方和接收方位于不同线程中时使用直接连接是不安全的，这与调用于另一个线程中的对象上的任何函数都是不安全的道理是一样的。



### 信号与槽机制的优点

类型安全：信号与槽的参数个数和类型要一致，但是槽的参数个数可以少于信号的参数个数，但缺少的参数必须是信号的参数的最后一个或者最后几个参数


松散耦合：激发信号的QT对象无须知道哪个对象的哪个函数接收信号，它只需要在适当的时间发送适当的信号。对象的槽也无须知道哪些信号关联了自己，互相独立



### 信号与槽的效率

信号与槽机制增强了对象间通信的灵活性，然而也损失了一些性能，运行速度慢了

- 需要定位接收信号的对象

- 安全的遍历所有关联(一个信号关联多个槽)

- 编组 / 解组 传递的参数

- 多线程的时候，信号可能需要排队等待



------------



2019-07-14 23:12:49 星期日