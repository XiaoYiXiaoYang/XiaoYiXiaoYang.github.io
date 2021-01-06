---
title: 数据结构
date: 2019-12-24 22:20:51
tags: [数据结构]
categories: [学习笔记]
---

 开始复习枯燥but重要的数据结构知识

<!--more-->

##### 数据结构
程序=数据结构+算法

数据：信息的载体
数据元素：数据的基本单位
数据项：构成数据元素的不可分割的最小单位
数据结构：相互之间存在一定关系的数据元素的集合
数据的逻辑结构：数据元素之间逻辑关系的整体（集合、线性结构、树结构、图结构）
数据的物理结构：数据的存储结构，数据及其逻辑结构在计算机中的表示（顺序存储结构、链接存储结构）

算法：（输入、输出、有穷性、确定性、可行性）
好算法：（正确性、鲁棒性、简单性、抽象分级、高效性）

**线性表的顺序存储结构----顺序表**
（随机存取结构）

```c++
const int Maxsize=100;
template<class DataType>
class SeqList
{

	public:
		SeqList(){length=0;}                //无参构造函数
		SeqList(DataType a[],int n);             //有参构造函数
		~SeqList(){}
		 int Length();        //求长度
		 DataType Get(int i);                  //按位查找
		 int Locate(DataType x);              //按值查找
		 void Insert(int i,DataType x);           //插入
		 DataType Delete(int i);                   //删除
         void PrintList();                     //遍历操作， 依次输出各元素
	 private:
		 DataType data[Maxsize];
		 int length;
};
```



无参构造--创建一个空的顺序表，length=0

有参构造，根据传入的n为数值长度，数组a为顺序表的每个元素



```c++
template<class DataType>
SeqList <DataType>::SeqList(DataType a[],int n)
{
	if(n>Maxsize)
	throw"参数非法";
	for(int i=0;i<n;i++)
	data[i]=a[i];
	length=n;
}
```



求线性表长度--直接返回length值
按位查找--根据下标i-1值取出第i元素
DataType SeqList<DataType>::Get(int i)
{
	if(i<1&&i>length)
	throw"位置非法";
	else
	cout<<data[i-1];
	return data[i-1];
 }

按值查找--根据元素值一一比对，直到相等，输出其值



```c++
template<class DataType>
int SeqList<DataType>::Locate(DataType x)
 {
 	for(int i=0;i<length;i++)
 	{
 	if(data[i]==x)
 	{
	cout<<i+1;
	}
 	if(i==length)
 	{
 	cout<<"没找到";
	 }
    }
 }
```



插入----从后到前，元素后移一位,然后在下标i-1插入元素x，length+1

```c++
template<class DataType>
void SeqList<DataType>::Insert(int i,DataType x)
 {
if(length>=Maxsize)
    throw"上溢";
	if(i<1||i>length+1)
	throw"位置";
	 for(int j=length;j>=i;j--)
	 data[j]=data[j-1];
	  data[i-1]=x;
	  length++;
}
```



删除--从i-1下标开始到尾，元素一一前移，length-1

```c++
template<class DataType>
DataType SeqList<DataType>::Delete(int i)
{
	if(length==0)
	throw"下溢";
	if(i<1||i>length)
	throw"位置";
	DataType x;
	x=data[i-1];
	for(int j=i;j<length;j++)
		data[j-1]=data[j];
	length--;
	return x;
}
```



遍历--从头到尾，一一输出

```
template<class DataType>
void SeqList<DataType>::PrintList()            //遍历输出
{
	for(int i=0;i<length;i++)
	cout<<data[i];
}
```

优点：随机存取
缺点：插入和删除要移动大量元素；表的容量难以确定；



**线性表的链接存储结构----链表**
单链表--结点由数据域，指针域组成

```c++
template<class DataType>
struct Node
{
    DataType data;
    Node<DataType> *next;
};
```



设置头指针指向第一个元素的所在结点，最后一个结点指针域为空即NULL，

```c++
template<class DataType>
class LinkList
{
public:
       LinkList();      					//无参数构造函数
       LinkList(DataType a[],int n);         //有参数构造函数
       ~LinkList();                        //析构函数
       int Length();                  //求单链表长度
       DataType Get(int i);           //按位查找
       int Locate(DataType x);              //按值查找
       void Insert(int i,DataType x);           //插入操作
       DataType Delete(int i);                //删除操作
       void PrintList();                        //遍历操作
private:
        Node<DataType> *first;                 //单链表的头指针
};
```



无参构造--初始化first指针next域为空

```c++
template<class DataType>
LinkList<DataType>::LinkList()          //无参构造
{
      first->next=NULL;
}
```



有参构造--尾插法--新插入的结点在终端结点后面
s = new node;s-data = a[i];
r->next = s; r = s;

```c++
template<class DataType>
LinkList<DataType>::LinkList(DataType a[],int n)       //有参构造函数
{
    Node<DataType> *s,*r;
    first=new Node<DataType>;
    r=first;
    for(int i=0; i<n; i++)
    {
        s=new Node<DataType>;
        s->data=a[i];
        r->next=s;
        r=s;
    }
    r->next=NULL;	//在最后将指向下一个的指针域置空
}
```



有参构造：头插法，新加入的结点在最前面

s = new node；s->data = a[i];

s->next = r; r = s;

```c++
template<class DataType>
LinkList<DataType>::LinkList(DataType a[],int n)       //有参构造函数
{
    Node<DataType> *s,*r;
    first=new Node<DataType>;
    first->next = NULL;
    r=first;
    for(int i=0; i<n; i++)
    {
        s=new Node<DataType>;
        s->data=a[i];
        s->next=r;
        r=s;
    }
    first = r;   //将头指针指向头结点
}
```



析构--从头到尾一一释放内存空间

```c++
template<class DataType>
LinkList<DataType>::~LinkList()                 //析构函数
{
      while(first!=NULL)
	  {
	  	Node<DataType> *q;
	  	q=first;                        //暂存被释放节点
	  	first=first->next;              //first指向被释放节点的下一个
	  	delete q;
	  }
}
```

求长度 --从头结点开始，只要next域不是NULL，数下去

```
template<class DataType>
int LinkList<DataType>::Length()                //求长度
{
	Node<DataType> *p;
     p=first->next;
	 int count=0;          //工作指针p和count初始化
     while(p!=NULL)
	{
     p=p->next;
     count++;
    }
     cout<<'\n'<<"线性表长度为\t"<<count<<endl;
     return count;
}
```

按位查找--从头结点开始遍历数到底i-1个结点

```
template<class DataType>
DataType LinkList<DataType>::Get(int i)                 //按位查找
{
	Node<DataType> *p;
     int count=1;
     p=first->next;
	              //工作指针p和count初始化
     while(p!=NULL&&count<i)
    {
        count++;
      p=p->next;                           //工作指针p后移

    }
    if(p==NULL)throw"位置";
    else
     cout<<"第"<<i<<"个位置的元素为："<<p->data;
	return p->data;
}
```


按值查找 --从头结点开始一一比较

```
template<class DataType>
int LinkList<DataType>::Locate(DataType x)              //按值查找
{
	Node<DataType> *p;
     p=first->next;
	int count=1;             //工作指针p和count初始化
     while(p!=NULL)
{
	if(p->data==x)
	{
	cout<<count;
	return count;
    }
     p=p->next;
     count++;
     }
}
```

插入 --找到第i-1位置，
s=new Node<DataType>;s->data=x;
s->next=p->next;p->next=s;

```
template<class DataType>
void LinkList<DataType>::Insert(int i,DataType x)             //插入
{
	Node<DataType> *p;
    p=first;
	int count=0;                //工作指针p指向头结点
    while(p!=NULL&&count<i-1)         //查找第i-1个节点
   {
      p=p->next;                           //工作指针p后移
      count++;
   }
      if(p==NULL)
	  throw"位置";             //没找到
      else
	  {
	  	Node<DataType> *s;
	    s=new Node<DataType>;
		s->data=x;           //申请一个节点s，其数据域为x
	    s->next=p->next;
		p->next=s;        //将结点s插入到结点p之后
	  }
}
```

删除 -- 找到第i-1个元素，摘链

```
template<class DataType>
DataType LinkList<DataType>::Delete(int i)          //删除
{
   Node<DataType> *p;
   p=first;
   int count=0;
   while(p!=NULL&&count<i-1)
   {
    p=p->next;
    count++;
   }
     if(p==NULL||p->next==NULL)
     throw"位置";
     else
	 {
    Node<DataType> *q;
     q=p->next;
     DataType x;
	 x=q->data;
     p->next=q->next;
     delete q;
     return x;
	 }
}
```

遍历--头结点开始，一一输出


**循环链表**（将终端节点next域指向头结点）
**带尾指针的循环链表**（将rear指向尾结点）

**双链表**（添加prior前驱指针域）

*问题思考：*今天想到一个问题，看似很简单，却让我绕进去的问题，线性表分为顺序表和单链表，顺序表很简单他是由数组来存储的，那么我的问题是，单链表是怎么存储的，为什么一个元素就能指向下一个元素呢，它的指针难道具有计一功能了？？？
事实是我基础太差了，忘记了单链表还需要结构体来描述单链表的结点，那个结点就是存储的东西，每次存储新的结点都要new 一个结构体，然后用该结构体的data域存放数据，用该结构体的next指针域存放指向下一个节点的地址，所以每次遍历结点的时候，就是在这些结构体结点之间遍历。

------------

*2019-03-15 15:32:23 星期五
萧逸小杨*

