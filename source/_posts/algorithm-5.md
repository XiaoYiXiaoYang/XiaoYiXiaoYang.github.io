---
title: 剑指offer（面试需要的基础知识）
date: 2019-12-15 20:03:31
tags: [算法]
categories: [学习笔记]
---

 名企面试官精讲典型编程题

<!--more-->



**面试的流程**

简单总结一下
技术面环节
1. 扎实的基础知识
1. 高质量的代码
1. 清晰的思路
1. 优化效率的能力
1. 优秀的综合能力

**面试需要的基础知识**

#### C++

4个类型转换的关键字
dynamic_cast
const_cast
static cast
reinterpret_cast

定义一个空的类型，里面没有任何成员变量和成员函数，对其类型求sizeof，得到结果是多少
**1**

注意：这个1不是this指针

声明类型的实例的时候，必须在内存中占有一定的空间，否则无法使用这些实例

```
class A{

};
int main()
{
    cout<<sizeof(A)<<endl;
    return 0;
}
输出：
1
```

在类型中添加一个构造函数和析构函数，再求sizeof，结果是
**1**
构造函数和析构函数只需要知道函数地址，与类的实例无关

```
class A{
    A(){}
    ~A(){}
};
int main()
{
    cout<<sizeof(A)<<endl;
    return 0;
}
输出：
1
```

如果把析构函数标记为虚函数，结果是
**4**
有虚函数，就会为该类型生成虚函数表，并在该类型的每一个实例中添加一个指向虚函数表的指针，一个指针占4字节

```
class A{
    A(){}
   virtual ~A(){}
};
int main()
{
    cout<<sizeof(A)<<endl;
    return 0;
}
输出：
4
```

简直神奇，接着来个一般性的

```
class A{
public:
    A(){}
private:
    int a;
    double b;
};
输出：
16
```



