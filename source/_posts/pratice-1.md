---
title: 浅拷贝踩内存
date: 2019-12-15 20:36:16
tags: [技术分享]
categories: [实习]
---

做UI界面，遇到了浅拷贝、赋值运算符问题

<!--more-->

#### 浅拷贝踩内存

UI使用QListWidget，向其中插入Item

每个Item是new出来的，超过一定数目就会delete最后的item



每个item可以点击详情

在详情页点击上一个下一个会将详情页的item发送至主窗口页去计算上一个或下一个item的位置，然后显示

为什么这样做，因为item是不断刷新的，数据会一直上来



1.点击一个item显示详情

2.当主窗口的Item不断刷新，将该item被delete后

3.在详情上点击上一个或者下一个

4.将详情的pitem发送至主窗口，主窗口通过QlistWidget的row函数访问会发生崩溃

```
int index = ui->listWidget->row(pitem);
```

我猜想row函数内部会取pitem的内容，导致地址虽然有效，但是内存已经被delete



#### 赋值操作





```
class A
{
A(int aa,int bb):a(aa),b(bb){}
	int a;
	int b;
}//产生实例的时候，可能部分值是未定义的
```





```c++
A a;

A aa(1,2);

aa = a;    //会发生什么
```



猜测：编译器会按照默认赋值运算符将每一个赋值过去



实战

```c++
class A
{
private:
	
public:
	A() { a = 66; }
	A(int aa,int bb):a(aa),b(bb) {}
	int a;
	int b;
};
int main()
{
	A a1;
	A a2(5,5);
	cout << a1.a << "---" << a1.b << endl;  //a是66 b是随机值
	cout << a2.a << "---" <<a2.b<< endl;    //a是5 b是5

	a2 = a1;
	cout << a2.a << "---" << a2.b << endl;
	system("pause");
	return 0;
}
```



结果输出 66  858993460



说明猜测正确

