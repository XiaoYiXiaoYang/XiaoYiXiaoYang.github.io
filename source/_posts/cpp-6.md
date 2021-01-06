---
title: C++基础（类的组合、复制构造函数）
date: 2019-12-08 19:13:32
tags: [C++]
categories: [学习笔记]
---

 分享复习组合类时遇到的大实验

<!--more-->



**类的组合**（一个类内嵌其他类的对象作为成员的情况，他们之间是一种包含与被包含的关系）

构造顺序
1.调用内前对象的构造函数，调用顺序按照内嵌兑现在组合类中的定义中出现的次序，（内嵌对象在构造函数的初始化列表中出现的顺序与内前对象构造函数的调用顺序无关）
2.执行本类构造函数的函数体



CPU：定义并实现一个CPU类。包含等级（rank）、频率（frequency）、电压(voltage)等属性；包含无参构造函数、有参构造函数、复制构造函数、析构函数、公有成员函数run、stop以及和属性对应的公有成员函数set、get。其中rank为枚举类型CPU_Rank，定义为enum CPU_Rank {P1=1, P2, P3, P4, P5, P6, P7}，frequency为单位为MHz的整型数，voltage为浮点型的电压值。 
定义并实现一个RAM类，包含无参构造函数、复制构造函数、析构函数和公有成员函数run、stop。 
定义并实现一个CDROM类，包含无参构造函数、复制构造函数、析构函数和公有成员函数run、stop。 
定义并实现一个简单的Computer类，包含芯片（cpu）、内存（ram）、光驱（cdrom）等属性，cpu为CPU类的一个对象，ram为RAM类的一个对象，cdrom为CDROM类的一个对象；包含有参构造函数、析构函数和公有成员函数run、stop。 
在主函数中，创建Computer对象，调用run、stop函数，观察构造函数和析构函数的调用顺序。



```c++
enum CPU_Rank {P1, P2, P3, P4, P5, P6, P7};//枚举类型
class CPU{
public:
	CPU(){ cout<<"调用了CPU无参构造函数"<<endl;}
	CPU(CPU_Rank Rank1, int Frequency,float Volrage){ cout<<"调用了CPU有参构造函数"<<endl; }
	CPU(CPU &p){cout << "调用了CPU复制构造函数"<<endl;}
	~CPU()     {cout << "调用了CPU析构函数"<<endl;}
	void Run() {cout << "调用了CPU类的run函数"<<endl;}
	void Stop(){cout << "调用了CPU类的stop函数"<<endl;}
	void Set(CPU_Rank Rank1, int Frequency,float Volrage) {cout << "调用了CPU类的set函数"<<endl;}
	void Get(CPU_Rank Rank1, int Frequency,float Volrage) {cout << "调用了CPU类的get函数"<<endl;}
private:
	CPU_Rank Rank1;     //rank
	int Frequency;     //NHZ
	float Voltage;    //电压
};

class RAM{
public:
	RAM(){cout << "调用了RAM无参构造函数"<<endl;}
	RAM(RAM &P){cout << "调用了RAM复制构造函数"<<endl;}
	~RAM(){cout << "调用了RAM析构函数"<<endl;}
	void Run(){cout << "调用了RAM类的run函数"<<endl;}
	void Stop(){cout << "调用了RAM类的stop函数"<<endl;}
private:
};

class CDROM{
public:
	CDROM(){cout << "调用了CDROM无参构造函数"<<endl;}
	CDROM(CDROM &p){cout << "调用了CDROM复制构造函数"<<endl;}
	~CDROM(){cout << "调用了CDROM析构函数"<<endl;}
	void Run(){cout << "调用了CDROM类的run函数"<<endl;}
	void Stop(){cout << "调用了CDROM类的stop函数"<<endl;}
private:

};

class Computer{
public:
	Computer(CPU xcpu,RAM xram,CDROM xcdrom);
	~Computer(){cout << "调用了Computer析构函数"<<endl;}
	void Run(){cout << "调用了Compter类的run函数"<<endl;}
	void Stop(){cout << "调用了Computer类的stop函数"<<endl;}
private:

	RAM ram;
	CDROM cdrom;
	CPU cpu;
};
Computer::Computer(CPU xcpu,RAM xram,CDROM xcdrom):cpu(xcpu),ram(xram),cdrom(xcdrom){
	cout << "调用了Computer有参构造函数\n";
}
int  main(){
	CPU x_cpu;
	RAM x_ram;
	CDROM x_cdrom;
	Computer computer1(x_cpu,x_ram,x_cdrom);
	computer1.Run();
	computer1.Stop();
	return 0;
} 
```



首先前三句分别创建了cpu，ram，cdrom对象，下面是computer对象的创建，computer构造函数是有参构造，先将已经创建的三个实参复制给构造函数的形参，复制顺序是形参表中从后往前，然后是computer类的数据成员的构造，因为它的数据成员是其他三个类对象，所以要复制构造这三个对象，构造顺序为computer类中数据成员声明的顺序，接下来computer构造函数就被调用，

构造输出结构：

```
CPU无参
RAM无参
CDROM无参

CDROM复制
RAM复制
CPU复制
CPU复制
RAM复制
CDROM复制
Computer有参
```



接着就要释放构造的形参，析构顺序相反，终于创建computer类对象操作完成。下面就是run函数，stop函数调用，到return 之后，就要开始析构computer类对想，析构顺序和构造顺序相反。


课本也提供了一个Point类构造Line类的代码，和这个类似，构造顺序一样，都需要理解记忆。



**前向引用声明**（在引用未定义的类之前，将该类的名字告诉编译器，使编译器知道那是一个类名）

尽管使用了前向引用声明，但是在提供一个完整的类定义之前，不能定义该类的对象，也不能在内联函数中使用该类的对象。



**构造函数**（在对象被创建时利用特定的值构造对象）
与类名相同，且没有返回值，
只要类中有了构造函数，编译器就会在对象被创建的时候被自动调用

如果没有构造函数，系统会生成默认无参构造函数



**复制构造函数**（使用一个已存在的对象，去初始化同类的一个新对象）

如果没有声明复制构造函数，系统会生成隐含的复制构造函数，它的功能是把初始值对象的每个数据成员的值否赋值到新建立的对象中

被调用的三种情况：
1.当用类的一个对象去初始化该类的另一个对象时
2.如果函数的形参是类的对象，调用函数时，进行形参和实参的结合
3.如果函数的返回值是类的对象，函数执行完成后返回调用者时



**析构函数**（完成对象被删除前的一些清理工作）

名字由类名前加“~”构成，没有返回值，析构函数不接受任何参数



------------

*2019-03-14 16:53:27 星期四
萧逸小杨*