---
title: C++ primer（第六章 函数）
date: 2019-12-14 22:41:52
tags: [C++ Primer]
categories: [学习笔记]
---

 函数的一个声明，重载函数，调用匹配

<!--more-->

#### 函数基础

调用运算符：一对圆括号，圆括号内是逗号分隔开的实参列表

函数调用
一实参初始化对应的形参
二控制权转移给被调函数

函数返回
一返回return语句中的值
二控制权从被调函数转移回主调函数

为了与C兼容，可以使用关键字void表示函数没有形参

返回类型（void，不能是数组或函数，可以是指向数组或函数的指针）

**局部对象** ：参数和函数体内部声明的变量是局部变量，局部变量还会隐藏外层作用域中同名的其他所有声明

**自动对象**：只存在于块作用域执行期间的对象称为自动对象

**局部静态对象**：在程序执行路径第一次经过对象定义语句时初始化，并且直到程序终止才被销毁

**函数声明**：函数原型，声明可以不要形参的名字，函数三要素（返回类型，函数名，形参类型）

**分离式编译**：把程序分割到几个文件中去，每个文件独立编译，


#### 参数传递

引用传递：形参是引用类型，它将绑定到对应的实参上去

值传递：实参的值被拷贝给形参，实参和形参是相互独立的对象

指针形参：当执行指针的拷贝操作时，拷贝的是指针的值，拷贝之后，两个指针是不同的指针，指针可以间接访问它所指对象，所以通过指针可以修改它所指对象的值

传引用参数：对于引用的操作实际是作用在引用所引的对象上

**使用引用避免拷贝**

拷贝大的类类型，或者容器对象比较低效

例：我们准备拷贝一个函数比较两个string对象的长度，因为string对象可能非常长，所以把形参定义成对常量的引用

```
bool isShorter(const string &r1,const string &r2)
```

**const形参和实参**

```
const int ci=42;   //不能改变ci，const是顶层的
int i=ci；         //正确，当拷贝ci时，忽略其顶层const
int *const p = &i;  //const是顶层的，不能给p赋值
*p =0; 				//正确，通过p修改对象是允许的
```

```
void fcn(const int i);
void fcn (int i);   //错误，调用时顶层const被忽略，参数一样   
```

**指针或引用形参与const**

非常量可以初始化底层const对象
底层const对象不能初始化非常量

```
int i=42;
const int *cp =&i;
const int &r = i;
const int &r2 =42;
int *p = cp;   //错误，类型不匹配
int &r3 =r;    //错误，类型不匹配
int &r4 =42;   //错误，不能用字面值初始化一个非常量引用
```

**尽量使用常量引用**

```
string::size_type find_char(string &s,char c,string::size_type &occurs)

find_char("hello world",'w',ctr);  //编译错误，第一个形参应该是const string &
```

**数组形参**

```
void print(const int*);
void print(const int[]);
void print(const int[10]);

等价
```

管理指针形参的三种办法

```
1.使用标记指定数组长度  
void print(const char *cp){
	if(cp)
		while(*cp)
}

2.使用标准库规范
void print(const int *beg,const int *end)

3.显式传递一个数组大小的形参
void print(int a[],size_t size)
```

**main:处理命令行选项**

```
prog -d -o ofile data0

int main(int argc,char *argv[])
```

argv是一个数组，他的元素是指向C风格字符串的指针

argc表示数组中字符串的数量

arg[0] = "prog"
arg[1] = "-d"
arg[2] = "-o"
arg[3] = "ofile"
arg[4] = "data0"
arg[5] = 0;

含有可变形参的函数

**initializer_list形参**

实参数量未知，但是全部实参类型都相同

```
void error_msg(initializer_list<string> il)
```

**省略符形参**

```
void foo(parm_list,...);
void foo(...);
```
在头文件varargs中

#### 返回类型和return语句

无返回值，在最后一句后面隐式的执行return
有返回值,return语句返回值的类型必须与函数的返回类型形同，或者能隐式转换成函数的返回类型

**值是如何被返回的**

```
string make(const string &word,const string &ending){
	return word+ending;
}
该函数返回word+ending的临时string对象
```

```
const string make(const string &word,string &ending){
	return word;
}
形参和返回类型都是const string的引用，调用函数和返回结果都不会真正拷贝string对象
```

**不要返回局部对象的引用或指针**

```
const string &manip(){
	string ret;
	if(!ret.empty())
		return ret;   //错误，返回局部对象的引用
	else
		return "empty";  //错误，字符串字面值转换成一个局部临时string对象，empty是一个局部临时量
}
```

**引用返回左值**

能为返回类型是非常量引用的函数的结果赋值
get_val(s,0) ='A';   //将S[0]的值该为A

如果返回类型是常量引用，就不能给调用的结果赋值


**列表初始化返回值**

return {}; //返回一个空vector对象
return {"aaa"};  //返回列表初始化的的vector对象

**主函数main的返回值**

如果到达main函数结尾却没有return语句，则编译器将隐式的插入一条返回0的return语句

cstdlib头文件中定义了两个预处理变量，分别表示成功失败

```
return EXIT_FAILURE;
return EXIT_SUCCESS;
```

**返回数组指针**

```
typedef int arr[10];   //arr是个别名，表示类型是含有10个整数的数组

using arr = int[10];  //另一种声明方式，等价
arr * func(int i); //func返回一个指向含有10个整数的数组的指针
```

```
int arr[10];
int (*p)[10]=&arr;   //p是一个指针，它指向含有10个整数的数组
```

```
Type (*function(parameter_list))[dimension]
int (*func(int i))[10]

func(int i)  //函数
*func(int i)  //对函数返回值解引用
*func(int i) [10]  //解引用得到大小为10的数组
int (*func(int i)) [10]   //数组中的元素值int类型
```

使用尾置返回类型

```
auto func(int i)->int(*)[10]
```

使用decltype

```
int odd[]={};
int even[]={};
decltype(odd) *arrptr(int i)

```


#### 函数重载

参数的个数、类型

**重载和const形参**

```
Record lookup(Phone);   //重复声明
Record lookup(const phone);

Record lookup(Phone*);   //重复声明
Record lookup(phone* const);
```
一个拥有顶层const形参无法和另一个没有顶层const的形参区分
如果形参是某种类型的指针或引用，则通过区分其指向的是常量对象还是非常量对象可以实现函数重载，此时const是底层的

```
Record lookup(Account&);         //非常量引用
Record lookup(const Account&);   //常量引用
```

const_cast和重载

```
string &shorterString(string &s1,string &s2)
{
	auto &r =shorterString(const_cast<const string &>(s1),const_cast<const string &>(s2));
	return const_cast<string &>(r);
}
```

首先将实参转换成对const的引用，再调用shorterString函数的const版本
const版本返回对const string的引用，再将其转换回一个普通的string &

重载调用
1.找到一个与实参最佳匹配的函数
2.找不到任何一个函数与调用的实参匹配，编译器发出无匹配的错误
3.有多于一个函数可以匹配，但是每一个都不是明显的最佳选择，产生错误，叫二义性调用

**重载与作用域**

```
string read();
void print(const string&);
void print(double);
void foo(int val){
	bool read =false;   //隐藏外层read
	string s =read();  //错误，read被隐藏
	
	void print(int);   //新的print，隐藏之前的print
	print("value");   //错误，print被隐藏
	print(val);       //正确 print(int)
	print(3.14);      //正确 print(int)
}
```

#### 特殊用途语言特性

**默认实参**

```
typedef string::size_type sx;
string screen(sz ht =24,sz wid =80,char backgrnd='')

string window;
window =screen();   //调用screen(24,80,'')
window = screen(60);  //调用sereen(60,80,'')
windos = screen('?')   //调用screen(63,80,'')   ?的ASCII值为63
```

只能省略尾部实参

**默认实参声明**

函数的后续声明只能为之前那些没有默认值的形参添加实参，而且该形参右侧的所有形参必须都有默认值

```
string screen(sz,sz,char='')

string screen(sz,sz,char='*')  //错误，重复声明，不能修改一个已经存在的默认值

string screen(sz,sz=,char='')
```

**默认实参初始值**
局部变量不能作为默认实参，除此之外，只要表达式的类型能转换成形参所需的类型，该表达式就能作为默认实参

```
sz wd=80;
char def='';
sz ht();
string screen(sz=ht(),sz=wd,char def);

void f2(){
def="*";
sz wd=100;
window =screen();  //调用screen(ht(),80,'*')
}
```

**内联函数和constexpr函数**

内联函数在调用点“内联的展开”

constexpr函数是指能用于常量表达式的函数

```
constexpr int new_size(){return 42;}

constexpr int foo =new_size();  //正确
```
把new_size()定义成无参constexpr函数，所以可用new_size()函数初始化constexpr类型的变量foo

**把内联函数和constexpr函数放在头文件内**

对于内联函数和constexpr函数的多个定义完全一致

**调试帮助**

**断言 assert 预处理宏**

assert(expr)	对expr求值，如果表达式为假，assert输出信息并终止程序的执行，如果表达式为真，assert什么也不做

assert宏定义在头文件cassert中

**NDEBUG预处理变量**

如果定义了NDEBUG，则assert什么也不做

assert是一种辅助调试的手段

```
void print(const int ia[],size_t size){
#define NDEBUG
	cerr<<_ _func_ _<<endl;
#endif
}

_ _func_ _   //输出当前调试的函数的名字
_ _FILE_ _   //存放文件名的字符串字面值
_ _LINE_ _   //存放当前行的整型字面值
_ _TIME_ _   //存放文件编译时间的字符串字面值
_ _DATE_ _   //存放文件编译日期的字符串字面值
```


#### 函数匹配

```
void f();
void f(int);
void f(int,int);
void f(double,double=3.14);
f(5.6);   //调用f(double，double)
```

**确定候选函数和可行函数**

第一步 选定本次调用中对应的重载函数集，集合中的函数为候选函数

第二步 从候选函数中选出能被这组实参调用的函数，选出的叫可行函数

可行函数特征
1.形参数量和实参对应
2.实参类型和对应形参类型相同

**寻找最佳匹配**

一、检查函数调用提供的实参，形参类型和实参类型越接近，他们匹配的越好

前面调用f(5.6)
如果调用f(int) 则需要将5.6转换为int
如果调用f(double,double) 则和实参精确匹配了

**含有多个形参的函数匹配**

- 该函数每个实参的匹配都不劣与其他可行函数需要的匹配
- 至少有一个实参的匹配优于其他可行函数提供的匹配

如果调用f(42,2.56)
则f(int,int)、f(double,double)都没有优势，编译器最终因为这个调用具有二义性而拒绝其请求

**实参类型转换**

1.精确匹配

- 实参类型和形参类型相同
- 实参从数组类型或函数类型转换成对应的指针类型
- 向实参添加顶层const或从实参中删除顶层const


2.通过const转换实现的匹配

3.通过类型提升实现的匹配

4.通过算术类型转换或指针转换实现的匹配

5.通过类类型转换实现的匹配


```
void f(int);
void f(short);
f('a');    //调用f(int),char提升成int
```

```
void f(long);
void f(float);
f(3.14);  		//错误，二义性调用
```


**const实参**

```
void f(Account&);
void f(const Account&);

const Account a;
Account b;
f(a);
f(b);
```

传入常量对象则调用f(const Account&)，
传入非常量对象则调用f(Account&)，虽然f(const Account&)也可以，但是需要类型转换


#### 函数指针


函数指针指向的是函数而非对象

```
bool length(string s1,string s2);

bool （*pf）length(string s1,string s2);   //括号不可少
bool *pf(string s1,string s2);   //如果不加括号，返回值是bool *

或者
pf = length;
pf = &length;   //取地址符可选
```

**重载函数的指针**

```
void f(int *);
void f(unsigned int);

void (*pf1) (unsigned int) =ff;  //正确，pf1指向ff(unsigned)
void (*pf2)(int) =ff; 	//错误，没有函数和该形参列表匹配
void (*pf3)(int *)=ff;   //错误，返回类型不匹配
```

编译器通过指针类型决定用哪个函数，指针类型必须与重载函数中的某一个精确匹配

**函数指针形参**


```
void use(bool pf(string s1,string s2));   //参数会自动转换为指向函数的指针
void use(bool (*pf) (string s1,string s2));  //显式的将形参定义成指向函数的指针

use(length);  //把函数当实参调，它会转换成指针
```

```
typedef bool func(string s1,string s2);
typedef decltype(func) func2;


typedef bool(*f)(string s1,string s2);
typedef decltype(length) *fp;
```
decltype返回函数类型，此时不会将函数类型自动转换成指针类型，因为decltype的结果是函数类型，所以只有在前面加上*才能得到指针


**返回指向函数的指针**

```
using F = int(int*,int);   //F是函数类型，不是指针
using PF = int(*)（int *,int）;   //pf是指针类型
```

将auto和decltype用于函数指针类型

```
size_type sumlength(string s1,string s2);
size_type large_length(string s1,string s2);

decltype(sumlength) *getfcn(string s);
```

牢记我们将decltype用于某个函数时，他返回函数类型而非指针类型，因此，我们显式的加上星号以表示我们需要返回指针







------------