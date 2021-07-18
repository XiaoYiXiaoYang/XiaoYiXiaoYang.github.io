---
title: 设计模式----单例模式
date: 2020-01-08 16:42:35
tags: [设计模式]
categories: [学习笔记]
---

<center>
单例模式
</center>

<!--more-->

#### 单例模式

```
public class Singleton{
  private Singleton(){}
  
 private static Singleton single = new Singleton();
 public static Singleton getIntance(){
 	return single;
 }
}

main(){
	Singleton obj1 = Singleton.getIntance();
	Singleton ob2 = Singleton.getIntance();
	
	if(obj1 == obj2){
		System.out.println("获取的是同一个实例");
	}
}
```

**思考**

为什么必须限制个数?

```
当存在多个实例时，势力之间互相影响，会产生意想不到的bug
如果我们确保只有一个实例，就可以在这个前提条件下放心编程
```

在何时创建生成这个唯一的实例

```
上述例子在第一次调用getIntance方法时，Singleton类会被初始化，
这个时候static的实例single会被初始化，生成了唯一的实例
```


多线程不安全的Singleton

```c++
public class Singleton{
  private Singleton(){}
  
 private static Singleton single = null;  //不创建实例
 public static Singleton getIntance(){
 	if(single == null){
		single =  new Singleton;
	}

	return single;
 }
}
```