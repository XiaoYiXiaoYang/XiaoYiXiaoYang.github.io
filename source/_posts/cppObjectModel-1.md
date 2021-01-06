---
title: C++对象模型(成员布局)
date: 2019-12-10 22:56:48
tags: [C++对象模型]
categories: [学习笔记]
---



加上封装，且布局成本没有增加

<!--more-->



坐标类型和坐标个数都参数化



```c++
template<class type, int dim>
class Point
{
public:
    Point();
    Point(type coords[dim])
    {
        for(int i =0; i< dim;i++)
        {
            _coords[i] = coords[i];
        }
    }
    
    type &operator[](int i)
    {
        assert(i < dim && i > 0);
        return _coords[i];
    }
    
  private:
    type _coords[dim];
};


inline template<class type,int dim>
    ostream operator <<(ostream &os,const Point<type,dim>&pt)
{
    os<<"(";
    
    for(int i =0; i< dim - 1;i++)
    {
        os<<pt[i];
    }
    
    os<<pt[dim-1];
    os<<")";
}
```



### C++对象模式



成员分为：static，非static

函数分为：static、非static、virtual



#### 简单对象模型

一个对象是一个表格，表格中每一项是一个slot，指向一个成员



#### 表格驱动对象模型

一个对象一个表格，表格内有一个成员函数表指针指向成员函数表；一个数据成员表指向数据成员表

成员函数表中每一项又是一个slot，指向具体的函数；数据成员表中每一项则是具体的成员



#### C++对象模型

非static数据成员放在对象内

static数据成员放在对象之外

static成员函数放在对象之外

非static成员函数放在对象之外

virtual成员函数地址放在vtbl中，vptr指针又放在对象中



（虚函数表的第一项通常指向该对象的RTTI）【runtime type identification】



#### 对象模型影响程序

```C++
X foobar()
{
    X xx;
    X *px = new X;
    
    xx.foo();	//虚函数
    px->foo();
    
    delete px;
    return xx;
}

可能在内部被转化为
    
void foobar(X &result)
{
    _result.X::X();		//_result 来取代local xx [NRV]
    
    px = _new(sizeof(X));
    if(px != 0)
    {
        px->X::X();
    }
    
    foo(_result);
    (*px->vtbl[2])(px);
    
    if(px != 0)
    {
        (*px->vtbal[1])(px);
        _delete(px);
    }
}
```



### 对象的差异



C++程序设计模式直接支持三种programming paradigms(程序设计范式)

1.程序模型

2.抽象数据类型模型

3.面向对象模型



```C++
Book:Library_materials

Book book;
things1 = book;     //things1 是 Library_materials 类型，会发生裁切
things.check_in();  //对象类型，编译期间确定

Library_materials &things = book;
things2.check_in();   //会调用book.check_in()  多态
```

只有使用指针(pointer)和引用(reference)的间接处理，才支持面向对象程序设计所需的多态性质。



*多态下，可能无法得知指针到底指向何种类型的对象；虽然无法预期，但是却是良好的行为。*【确实耐人寻味】



非static的派生行为以及类型为void*的指针可以说是多态的，但他们并没有被语言明确的支持，也就是他们必须要程序员通过显式的转换操作来管理



支持多态的方法

1.由隐式的转换操作，使得基类的指针或引用指向派生类的对象

2.虚函数

3.dynamic_cast运算符typeid运算符



**需要多少内存表现一个对象**

1.其非static数据成员的总和

2.加上由于对齐需要而填补上去的空间

3.加上支持虚函数而由内部产生的额外负担



#### 指针的类型



**“指向不同类型的指针”，既不在其指针表示法不同，也不在其内容不同，而是在其寻址出来的对象的类型不同**



独立的类对象布局

ZooAnimal类

|      int loc       |
| :----------------: |
|  int String::len   |
| char * String::str |
|  _vptr_ZooAnimal   |



派生类Bear

|      int loc       |
| :----------------: |
|  int String::len   |
| char * String::str |
|  _vptr_ZooAnimal   |
|     int height     |
|   int cell_block   |



```C++
Bear b;
ZooAnamal *pz = &b;
Bear *pb = &b
```

差别：虽然都指指向对象b，但是pb指针涵盖的地址包含整个Bear对象，而pz指针所涵盖的地址只包含Bear对象中的ZooAnimal subobject



例

```C++
pz->foo();  //多态
```

多态执行时，pz所指的类型可以决定foo调用的实例，但是类型信息并不是维护于pz之中，而是维护于link之中，此link存在于“object 的vptr" 和 ”vptr所指的virtual table“之间



例

```C++
Bear b;
ZooAnimal za = b;		//切割
	
za.foo();			//将会调用ZooAnimal的
```

编译器必须确保如果某个对象含有一个或者一个以上的vptrs，那些vptrs的内容不会被基类对象初始化或改变

