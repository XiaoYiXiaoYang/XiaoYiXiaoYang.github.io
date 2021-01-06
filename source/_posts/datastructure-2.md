---
title: 数据结构（栈和队列）
date: 2019-12-25 10:35:19
tags: [数据结构]
categories: [学习笔记]
---

 分享学习栈和队列这两个操作受限的线性表

<!--more-->



#### 栈----后进先出

**顺序栈**

```c++
const int StackSize=10;
template<class DataType>
class SeqStack
{
public:
   SeqStack(){top=-1;}         //构造函数，初始化一个空栈
   ~SeqStack(){}           //析构函数为空
    void Push(DataType x);    //入栈操作，将元素x入栈
	DataType Pop();         //出栈操作，将栈顶元素弹出
    DataType GetTop(){if (top!=-1) return data[top];else return -1;}     //取栈顶元素
	int Empty();   //判断栈是否为空
private:
    DataType data[StackSize];    //存放栈顶元素的数组
	int top;     //栈顶指针 
 };
```



入栈----top指针加1，然后data[top]为x，

```c++
template<class DataType>
void SeqStack<DataType>::Push(DataType x)      //入栈 
{
	if(top==StackSize-1)
	throw"上溢";
	data[++top]=x; 
}
```



出栈----top-1，是否释放top下标的元素

```c++
template<class DataType>
DataType SeqStack<DataType>::Pop()     		//出栈
{
	DataType x; 
	if(top==-1)
	throw"下溢";
	x=data[top--];
	cout<<x<<endl;
	return x;	 
}
```



**两栈共享空间**

用一个比较大的数组存储两个栈，一个栈的栈底为该数组的始端，另一个栈的栈底为数组的末端

构造 top1 = -1；top2 = stacksize；
入栈  栈满为top2 = top1+1  入栈1 top1+1   入栈2 top2-1 
出栈  栈1空 top1=-1  top2=stacksize  出栈1 top1-1   出栈2 top2+1



**链栈**
将头结点作为栈顶,入栈则向栈顶加入元素，栈底元素的next域一直为NULL

top指向栈顶元素

链栈入栈
s = new node; s->data=x;
s->next = top;top = s;



链栈出栈
x = top->data; p=top; //暂存栈顶元素
top = top->next;      //将栈顶元素摘链



#### 队列----先进先出

允许插入的一端为队尾（入队），允许删除的一端为队头（出队）

**循环队列**
队头指针front指向队头元素的前一个位置，队尾元素指向队尾元素
随着队列插入操作，整个队列向数组中下标较大的位置移过去，从而产生了队列的“单向移动性”。当元素被插入到数组中下标最大的位置上后，队列的空间就用尽了，尽管此时数组的低端还有空闲空间，这叫“**假溢出**”

允许队列直接从数组中下标最大的位置延续到下标最小的位置，队列的头尾相接的顺序存储结构称为循环队列。
队空的条件--front =rear
队满的条件--front = (rear+1)%Queuesize = front

构造--初始化一个空的循环队列，只需将队头指针和队尾指针同时指向数组的**某一位置**，一般是front = rear =Queuesize-1
入队--将队尾指针rear在循环意义下+1，然后将带插入元素x插入队尾位置
出队--将front指针在循环意义下+1



**链队列**（队头指针front指向队列的头结点，队尾指针rear指向尾结点）

```c++
template<class DataType>
struct Node
{
	DataType data;
	Node<DataType> *next;
};
```



```c++
template<class DataType>
class LinkQueue
{
public:
	LinkQueue();              //构造函数，初始化一个空的链队列
	~LinkQueue(){};             //析构函数，释放链队列中各节点的存储空间
	void EnQueue (DataType x);          //入队操作，将元素x入队
	DataType DeQueue();                  //出队操作，将队头元素出队
	DataType GetQueue(){if(front==rear) return -1;else return front->next->data;}               //取链队列的队头元素
	int Empty();        //判断队列是否为空
private:
	Node<DataType>*front,*rear;            //队头和队尾指针，分别指向头节点和终端节点
};
```



构造--创建一个头结点s，s->next=NULL，将队头指针和队尾指针都指向s

```
Node<DataType>*s;
	s=new Node<DataType>;
	s->next=NULL;             //创建一个头节点s
	front=rear=s;             //将队头指针和队尾指针都指向头节点s
```



入队--将结点s插入队尾

```c++
    Node<DataType>*s;
	s=new Node<DataType>;
	s->data=x;        //申请一个数据域为x的结点s
    rear->next=s;      //将结点s插入队尾
	rear=s; 
	s->next=NULL; 
```



出队--队头元素为front->next，将它摘链

```
	p=front->next;
	x=p->data;         //暂存队头元素
	front->next=p->next;      //将队头元素所在结点摘链
```



**测试**：输入队列35412
取队头元素，返回3
出队3个元素，取对=队头元素为1
结果正确

------------

*2019-03-16 21:30:49 星期六
萧逸小杨*