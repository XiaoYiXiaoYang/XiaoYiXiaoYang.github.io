---
title: 智能指针及其内存泄漏
date: 2019-12-14 22:57:21
tags: [C++]
categories: [技术分享]
---

 shared_ptr、auto_ptr、weak_ptr、unique_ptr、内存泄漏怎么破

<!--more-->

### 智能指针

**auto_ptr**


```
auto_ptr< string> p1 (new string ("I reigned lonely as a cloud.”));
auto_ptr<string> p2;
p2 = p1; //auto_ptr不会报错.

```

p2剥夺了p1的所有权，但是当程序运行时访问p1将会报错。所以auto_ptr的缺点是：存在潜在的内存崩溃问题！



**unique_ptr**

同一时间只能有一个一个智能指针可以指向该对象

```
unique_ptr<string> p3 (new string ("auto"));   //#4
unique_ptr<string> p4；                       //#5
p4 = p3;//此时会报错！！
```

编译器认为p4=p3非法，避免了p3不再指向有效数据的问题。因此，unique_ptr比auto_ptr更安全。



当程序试图将一个 unique_ptr 赋值给另一个时，如果源 unique_ptr 是个临时右值，编译器允许这么做；



```
unique_ptr<string> pu3;
pu3 = unique_ptr<string>(new string ("You"));   // 允许
```

允许是因为表达式不会留下悬挂的unique_ptr，
因为它调用 unique_ptr 的构造函数，该构造函数创建的临时对象在其所有权让给 pu3 后就会被销毁。这种随情况而已的行为表明，unique_ptr 优于允许两种赋值的auto_ptr 


**shared_ptr**


shared_ptr实现共享式拥有概念。

多个智能指针可以指向相同对象，该对象和其相关资源会在“最后一个引用被销毁”时候释放。

从名字share就可以看出了资源可以被多个指针共享，它使用计数机制来表明资源被几个指针共享。

可以通过成员函数use_count()来查看资源的所有者个数。除了可以通过new来构造，还可以通过传入auto_ptr, unique_ptr,weak_ptr来构造。

当我们调用release()时，当前指针会释放资源所有权，计数减一。当计数等于0时，资源会被释放。




实现：核心要理解引用计数，什么时候销毁底层指针，还有赋值，拷贝构造时候的引用计数的变化，析构的时候要判断底层指针的引用计数为0了才能真正释放底层指针的内存

```
template <typename T>
class SmartPtr
{
private:
	T *ptr;    				//底层真实的指针
	int *use_count;			//保存当前对象被多少指针引用计数
public:
	SmartPtr(T *p); 				//SmartPtr<int>p(new int(2));
	SmartPtr(const SmartPtr<T>&orig);//SmartPtr<int>q(p);
	SmartPtr<T>&operator=(const SmartPtr<T> &rhs);//q=p
	~SmartPtr();

	T operator*();  //为了能把智能指针当成普通指针操作定义解引用操作
	T*operator->();  //定义取成员操作
	T* operator+(int i);//定义指针加一个常数
	int operator-(SmartPtr<T>&t1,SmartPtr<T>&t2);//定义两个指针相减
```



```
template <typename T> 
void getcount() 
{
	return *use_count 
}

template <typename T> 
int SmartPtr<T>::operator-(SmartPtr<T> &t1, SmartPtr<T> &t2) 
{ 
	return t1.ptr-t2.ptr;
}

template <typename T> 
SmartPtr<T>::SmartPtr(T *p) 
{ 
	ptr=p; 
	try {
		use_count=new int(1); 
	}catch (...){
		delete ptr;    //申请失败释放真实指针和引用计数的内存
		ptr= nullptr; 
		delete use_count; 
		use_count= nullptr;
	} 
}

template <typename T> 
SmartPtr<T>::SmartPtr(const SmartPtr<T> &orig) //复制构造函数
{
use_count=orig.use_count;//引用计数保存在一块内存，所有的SmarPtr对象的引用计数都指向这里
this->ptr=orig.ptr;
++(*use_count);//当前对象的引用计数加1
}

template <typename T> 
SmartPtr<T>& SmartPtr<T>::operator=(const SmartPtr<T> &rhs) {
//重载=运算符，例如SmartPtr<int>p,q; p=q;这个语句中，首先给q指向的对象的引用计数加1，因为p重新指向了q所指的对象，所以p需要先给原来的对象的引用计数减1，如果减一后为0，先释放掉p原来指向的内存，然后讲q指向的对象的引用计数加1后赋值给p

	++*(rhs.use_count); 
	if((--*(use_count))==0) 
	{
		delete ptr;
		ptr= nullptr;
		delete use_count;
		use_count= nullptr;
	}
	ptr=rhs.ptr;
	*use_count=*(rhs.use_count);
	return *this;
}


template <typename T> 
SmartPtr<T>::~SmartPtr() 
{ 
	getcount();
	//SmartPtr的对象会在其生命周期结束的时候调用其析构函数，在析构函数中检测当前对象的引用计数是不是只有正在结束生命周期的这个SmartPtr引用，如果是，就释放掉，如果不是，就还有其他的SmartPtr引用当前对象，就等待其他的SmartPtr对象在其生命周期结束的时候调用析构函数释放掉
	if(--(*use_count)==0)  
	{
		getcount();
		delete ptr;
		ptr= nullptr;
		delete use_count;
		use_count=nullptr;
	}
}

template <typename T>
T SmartPtr<T>::operator*()
{
	return *ptr;
}

template <typename T>
T*  SmartPtr<T>::operator->()
{
	return ptr;
}

template <typename T>
T* SmartPtr<T>::operator+(int i)
{
	T *temp=ptr+i;
	return temp;
}

```




**weak_ptr**

weak_ptr 是一种不控制对象生命周期的智能指针, 它指向一个 shared_ptr 管理的对象. 

进行该对象的内存管理的是那个强引用的 shared_ptr. weak_ptr只是提供了对管理对象的一个访问手段。

weak_ptr 设计的目的是为配合 shared_ptr 而引入的一种智能指针来协助 shared_ptr 工作, 它只可以从一个 shared_ptr 或另一个 weak_ptr 对象构造, 它的构造和析构不会引起引用记数的增加或减少.


weak_ptr是用来解决shared_ptr相互引用时的死锁问题,如果说两个shared_ptr相互引用,那么这两个指针的引用计数永远不可能下降为0,资源永远不会释放。

它是对对象的一种弱引用，不会增加对象的引用计数，和shared_ptr之间可以相互转化，shared_ptr可以直接赋值给它，它可以通过调用lock函数来获得shared_ptr。


```
class B;
class A
{
public:
shared_ptr<B> pb_;
~A()
{
cout<<"A delete\n";
}
};
class B
{
public:
shared_ptr<A> pa_;
~B()
{
cout<<"B delete\n";
}
};
void fun()
{
shared_ptr<B> pb(new B());
shared_ptr<A> pa(new A());
pb->pa_ = pa;
pa->pb_ = pb;
cout<<pb.use_count()<<endl;
cout<<pa.use_count()<<endl;
}
int main()
{
fun();
return 0;
}
```

可以看到fun函数中pa ，pb之间互相引用，两个资源的引用计数为2，当要跳出函数时，智能指针pa，pb析构时两个资源引用计数会减一，但是两者引用计数还是为1，导致跳出函数时资源没有被释放（A B的析构函数没有被调用）

如果把其中一个改为weak_ptr就可以了，我们把类A里面的shared_ptr pb_; 改为weak_ptr pb_; 

运行结果如下，这样的话，资源B的引用开始就只有1，当pb析构时，B的计数变为0，B得到释放，B释放的同时也会使A的计数减一，同时pa析构时使A的计数减一，那么A的计数为0，A得到释放。



### 智能指针内存泄漏

当两个对象相互使用一个shared_ptr成员变量指向对方，会造成循环引用，使引用计数失效，从而导致内存泄漏


解决循环引用导致的内存泄漏，引入了weak_ptr弱指针，weak_ptr的构造函数不会修改引用计数的值，从而不会对对象的内存进行管理，其类似一个普通指针，但不指向引用计数的共享内存，但是其可以检测到所管理的对象是否已经被释放，从而避免非法访问











2019-11-26 10:31:26 星期二