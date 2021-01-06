---
title: C++ primer（第七章 类的其他特性）
date: 2019-12-14 22:46:16
tags: [C++ Primer]
categories: [学习笔记]
---

 我们使用类定义自己的数据类型，通过定义新的类型来反映待解决问题中的各种概念

<!--more-->

**定义抽象数据类型**

数据抽象：依赖于接口和实现分离的编程技术

类的接口：包括用户所能执行的操作

类的实现：包括类的数据成员，负责接口实现的函数体以及定义类所需的各种私有函数

封装：实现了累的接口和实现的分离，封装后的类隐藏了它的实现细节，类的用户只能使用接口而无法访问实现部分


定义类

```
class Sales_data{
	string::isbn() const{return bookNo;}
	Sales_data &combine(const Sales_data&);
	double avg_price() const;
	
	string bookNo;
	ungigned units_sold=0;
	double revenue = 0.0;
};
```

通过成员访问运算符来做到这一点，因为this所指的正是这个对象，任何对类成员的直接访问都被看做this的隐式调用，相当于书写了this->bookNo

**引入const成员函数**

isbn()函数参数列表后的const关键字，作用是修改隐式this指针的类型

(默认情况下)，this指针是指向类类型非常量版本的**常量指针**(指针本身是常量)

```
Sales_data *const
```

(默认情况下)我们不能把this指针绑定到一个常量对象上，也不能在一个常量对象上调用普通的成员函数

C++语言允许把const关键字放在成员函数的参数列表之后，此时，紧跟在参数列表后面的const表示this是一个**指向常量的指针**，像这样的成员函数被称为常量成员函数

**类作用域和成员函数**

编译器分两步处理类，首先编译成员的声明，然后才轮到成员函数体，因此，成员函数题可以*随意使用类中的其他成员而无须在意这些成员出现的次序*。

**定义一个返回this对象的函数**

函数combine的设计初衷类似于复合赋值运算符+=

```
Sales_data& Sales_data::combine(const Sales_data &rhs)
{
	unit_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}
```

内置的赋值运算符把他的左侧运算对象当成左值返回，因此为了与它保持一致，combine函数必须返回*引用类型*，返回Sales_data&

return语句时解引用this指针以获得执行该函数的对象


**定义类相关的非成员函数**

```
istream &read(istream &is,Sales_data &item){
	double price=0;
	is >> item.bookNo >> item.units_sold >>price;
	item.revenue = price *item.units_sold;
	return is;
}
```
read函数从给定流中将数据读到给定的对象里

**构造函数**

构造函数不能声明成const的，当我们创建一个const对象时，直到构造函数完成初始化过程，对象才能真正取得其“常量”属性，因此，构造函数可以在const对象的构造过程中可以向其写值

**默认构造函数**

1.编译器只有在发现类不包含任何构造函数的情况下才会替我们生成一个默认构造函数，
2.对于某些类，合成的默认构造函数可能执行错误的操作(比如类內成员变量为数组或指针)
3.有时候编译器不能为某些类合成默认构造函数，(如果类中包含一个其他类类型的成员且这个成员的类型没有默认构造函数)

**=default**

```
Sales_data()=default;
```

C++11新标准，可以通过在参数列表后面写上=default来要求编译器生成构造函数

**构造函数初始值列表**

```
Sales_data(const string &s):bookNo(s){}
```

**友元**

类可以允许其他类或者函数成为他的友元


**类成员**

```
class Screen{
public:
	typedef string::size_type pos;
private:
	pos cursor=0;
	pos height=0,width=0;
	string contents;
};
```

**可变数据成员**

我们希望能修改类的某个数据成员，即使是在一个const成员函数内。

```
class Screen{
public:
	void some_member()const;
private:
	mutable size_t access_ctr;
};
```

一个可变数据成员永远不会是const，即使它是const对象的成员，因此，一个const成员函数可以改变一个可变成员的值


**返回星this的成员函数**

```
Screen &set(char c){
	contents[sursor] = c;
	return *this;
}


MyScreen.move(4,0).set('#');
```

如果我们令move返回Screen而非Screen&,则上述语句的行为将大不相同

```
Screen temp = myScreen.move(4,0);
temp.set('#');  //改变的是临时副本对象
```

**从const成员函数返回星this**

添加函数disply，负责打印Screen的内容，不需要修改内容，所以令display为一个const成员，此时this将是一个指向const的指针，而星this是const对象。由此推断，display的返回类型是const Sales_data&，然而这样不能把display嵌入其他动作。

```
myScreen.display(cout).set('#');
```

一个const成员函数如果以引用的形式返回星this，那么它的返回类型将是常量引用

**基于const的重载**

```
Screen &display(ostream &s)
Screen &display(ostream &s)const
```

即使两个类的成员列表完全一致，它们也是不同的类型，

**前向声明**

class Screen;

在它声明之后定义之前是一个不完全类型，不完全类型可以定义指向这种类型的指针或引用，也可以声明以不完全类型作为参数或返回类型的函数


**友元再探**

一个类指定了友元类，则友元类的成员函数可以访问此类包括非公有成员在内的所有成员。

令某个函数作为友元，则需要仔细组织程序的结构以满足声明和定义的彼此依赖关系


1.定义Window_mgr类，声明clear函数
2.定义Screen，包括对于clear的声明
3.定义clear

如果一个类想把一组重载函数生命成它的友元，它需要对这组函数中的每一个分别声明


**友元声明和作用域**

当一个名字第一次出现在一个友元声明中时，我们隐式的假定该名字在当前作用域中是可见的，然而，友元本身不一定真的声明在当前作用域中

```
struct X{
	friend void f(){}	  //友元函数可以定义在类的内部
	X(){ f();}            //错误，f还没声明
	void g();
	void h();
};
void X::g(){return f();}    //错误，f()还没被声明
void f();					//声明那个定义在X中的函数
void X::h(){return f();}     //正确
```

**类的作用域**

一个类就是一个作用域

在类的外部定义成员函数时必须同时提供类名和函数名，在类的外部，成员的名字被隐藏起来了

一旦遇到了类名，定义的剩余部分就在类的作用域之内。

```
void Window_mgr::clear(ScreenIndex i){
	Screen &s = screen[i];
	s.contents = string(s.height*s.width,' ');
}
```

另一方面，函数的返回类型通常出现在函数名之前，因此需要类名作用域指定

**名字查找与类的作用域**

首先，在名字所在的块中寻找其声明语句，只考虑在名字的使用之前出现的声明
如果没找到，继续查找外层作用域
如果最终没有找到匹配的声明，则程序报错


**成员定义中普通块作用域的名字查找**

首先，在成员函数内查找该名字的声明，只有在函数使用之前出现的声明才被考虑
如果在成员函数内没有找到，则在类内继续查找，这时类內的所有成员都可以被考虑
如果类內也没有找到该名字的声明，在成员函数定义之前的作用域内继续查找

**类作用域之后，在外围的作用域中查找**

如果编译器在函数和类的作用域中都没有找到名字，它将接着在外围的作用域中查找

```
void Screen::dummpy(pos height){
	cursor = width *::height;		//指的是全局的height
}
```

**构造函数再探**

成员初始化的顺序

第一个成员被初始化，然后第二个，然后第三个...
构造函数的初始值列表中初始值的前后位置关系不会影响实际的初始化顺序

```
class X{
int i;
int j;
public:
	X(int val):j(val),i(j){}  //未定义的
}
```

实际上i先被初始化，用j定义了i，所以i未定义


**默认实参和构造函数**

```
class Sales_data{
public:
	Sales_data(string s=""):bookNo(s){}
}
```

当没有给定实参或者给定一个string实参时，两个版本的类创建的相同的对象

**委托构造函数**

成员初始值列表只有一个唯一入口，就是类名本身
类名后面紧跟圆括号括起来的参数列表，参数列表必须与类中另外一个构造函数匹配

```
Sales_data(string s,unsigned cnt,double price):bookNo(s),units_sold(cnt),revenue(cnt*price){}
Sales_data():Sales_data("",0,0){}
Sales_data(string s):Sales_data(s,0,0){}
Sales_data(istream &s):Sales_data(){ read(s,*this);}
```

第一个构造函数接收三个实参，使用实参初始化数据成员，然后结束工作
第二个构造函数为默认构造函数，委托三参数构造函数
第三个构造函数接收一个string实参，委托三参数构造函数
第四个构造函数接收一个istream实参，委托*默认构造函数*，先执行默认构造函数，再执行read()函数




**隐式的类类型转换**

只允许一步类类型转换

```
string null_book="9999";
item.combine(null_book);


item.combine("9999");  //错误，两步转换了一步转为string，再一步转istream
```

**抑制构造函数的隐式转换**

将构造函数声明为explicit加以阻止

```
explicit Sales_data(const string &s):bookNo(s){ }
```

关键字explicit只对一个实参的构造函数有效，
需要多个实参的构造函数不能用于执行隐式转换，所以无需explicit指定


**explicit只能用于直接初始化**

```
Sales_data item1(null_book);   //正确
Sales_data item2 = null_book;   //错误
```

标准库中含显式构造函数的类

```
接受一个单参数的const char *string的构造函数不是explicit的
接受一个容量参数的vector的构造函数是ecplicit的
```

**聚合类**

所有成员是public的
没有定义任何构造函数
没有类内初始值
没有基类，也没有virtual函数，

```
struct Data{
	int val;
	string s;
}
```

我们可以提供一个花括号括起来的成员初始值列表，初始值顺序与列表顺序一直

Data val1 = {0,"Anna"};


**字面值常量类**

数据成员都是字面值类型的聚合类是字面值常量类

如果不是一个聚合类，需要满足以下条件，才是一个字面值常量类
1.数据成员都是字面值类型
2.类必须至少含有一个constexpr的构造函数
3.如果一个数据成员含有类內初始值，则内置类型成员的初始值必须是一条常量表达式或者如果成员属于某种类类型，则初始值必须使用成员自己的constexpr构造函数
4.类必须使用析构函数的默认定义

尽管构造函数不能是const的，但是字面值常量类的构造函数可以是constexpr函数

```
class Debug{
public:
	constexpr Debug(bool b=true):hw(b),io(b),other(b){}
private:
	bool hw;
	bool io;
	bool other;
}
```

**类的静态成员**

类的静态成员函数不与任何对象绑定在一起，它们不包含this指针，静态成员函数不能声明成const的，而且我们也不能在static函数体内使用this指针。

静态成员的类內初始化

即使一个常量静态数据成员在类內被初始化了，通常情况下也应该在类的外部定义一下该成员，



```
class Bar{
public:

private:
	static Bar mem1;
	Bar *mem2;
	Bar mem3;      //错误，数据成员必须是完全类型
}
```

静态成员和普通成员的另外一个区别是我们可以使用静态成员作为默认实参

非静态数据成员不能作为默认实参，因为它的值本身属于对象的一部分


```
class Screen{
public:
	Screen &clear(clear=bkgroud);
peivate:
	static const char bkgroud;
}
```



2019-06-29 12:03:24 星期六