---
title: 一堆面试准备
date: 2019-12-24 20:25:50
tags: [C++]
categories: [技术分享]
---

杂谈

<!--more-->



#### C++

1.STL源码中的traits

traits是类型识别技术

```c++
template<typename Iterator>  
void func(Iterator iter)  
{  
    *Iterator var;//这样定义变量可以吗？  
}  

template<typename Iterator, typename T>  
void func_impl(Iterator iter, T t)  
{  
    T temp;//这里就解决了问题  
    //这里做原本func()的工作  
}  
  
template<typename Iterator>  
void func(Iterator iter)  
{  
    func_impl(iter, *iter);//func的工作全部都移到func_impl里面了  
}  
```

通过内嵌类型声明，解决了参数是迭代器的情况



```
template<typename Iterator>  
(*Iterator) func(Iterator iter)  
{  
    //这样定义返回类型可以吗？  
}  

template<typename T>  
class Iterator  
{  
public:  
    typedef T value_type;//内嵌类型声明  
    Iterator(T *p = 0) : m_ptr(p) {}  
    T& operator*() const { return *m_ptr;}  
    //...  
  
private:  
    T *m_ptr;  
};  
  
template<typename Iterator>  
typename Iterator::value_type  //以迭代器所指对象的类型作为返回类型，长度有点吓人！！！  
func(Iterator iter)  
{  
    return *iter;  
}  

```

通过内嵌类型声明，解决了返回值是迭代器的情况



2.map底层架构是什么

红黑树

- 性质1：每个节点要么是黑色，要么是红色。
- 性质2：根节点是黑色。
- 性质3：每个叶子节点（NIL）是黑色。
- 性质4：每个红色结点的两个子结点一定都是黑色。
- **性质5：任意一结点到每个叶子结点的路径都包含数量相同的黑结点。**



3.知道kmp吗，kmp时间复杂度是多少，为什么从那开始前面的不用去查了





4.map插入有哪几种方法，insert方法的返回值（不知道这个）





5.类中的列表初始化和函数体中初始化有什么区别

C++构造函数体内初始化与列表初始化的区别：
结论是：
若某个类（下文的class B）有一个类成员是类类型（下文的class A），那么
1、若类B通过构造函数体内初始化，会先调用类A的默认构造函数（无参构造函数），再调用类A的赋值运算符；
2、若类B通过初始化列表去初始化，则只调用类A的拷贝构造函数。

6.多线程编程的一些函数join，detach作用

join是阻塞当前线程，并等待object对应线程结束，该线程继续执行搜索 
detach是将线程从当前线程分离出去，即不受阻塞，操作系统会将其独立对待

7.多态实现

基类的指针指向派生类的对象



8.析构函数为什么为虚函数

基类的指针指向派生类的对象，析构的时候不会析构，需要是虚函数



9.类什么函数不能为虚函数

静态成员函数

内联函数

普通函数

友元函数

构造函数



10.浏览器输入一个URL会发生什么

DNS域名解析协议 访问本地NDS服务器缓存，如果没有则让DNS服务器去访问  递归查询  迭代查询

http

tcp

ip

数据链路

物理层



11.什么时候需要重新定义拷贝构造函数

需要的时候

具体就是编译器合成的不合理的时候

编译器没合成 而编程需要的时候



12.I/O模型有哪些



13.malloc最大多少

地址空间限制是有的，但是malloc通常情况下申请到的空间达不到地址空间上限。内存碎片会影响到你“一次”申请到的最大内存空间。比如你有10M空间，申请两次2M，一次1M，一次5M没有问题。但如果你申请两次2M，一次4M，一次1M，释放4M，那么剩下的空间虽然够5M，但是由于已经不是连续的内存区域，malloc也会失败。系统也会限制你的程序使用malloc申请到的最大内存。Windows下32位程序如果单纯看地址空间能有4G左右的内存可用，不过实际上系统会把其中2G的地址留给内核使用，所以你的程序最大能用2G的内存。除去其他开销，你能用malloc申请到的内存只有1.9G左右。





