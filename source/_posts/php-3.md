---
title:  PHP面对象编程部分
date: 2020-01-08 11:22:48
tags: [PHP]
categories: [学习笔记]
---

 PHP面对象编程部分

<!--more-->

面向对象编程（OOP）把编程的重心从处理过程转移到了对现实世界的表达，面向对象编程具有三大特点，封装性、继承性、多态性。

- 类的声明- 

```
<?php
	权限修饰符 class 类名{
	类的内容；
	}
?>
```
public、protected、private
属性和方法默认项是public，这意味着属性和方法的各个项，从类的内部和外部都可以访问
用关键字private声明的属性和方法，则只能从类的内部访问，
用关键字protected声明的属性和方法，也只能从类的内部访问，**但是通过继承而产生的子类，也是可以访问这些属性和方法的**

成员属性  需要使用关键词修饰，如果没有特定的意义，仍然要使用var关键词修饰
成员方法
类的实例化--对象
类用来描述具有相同的数据结构和特征的“一组”对象，类是“对象的抽象”；“对象”是类的具体实例。
$变量名 = new 类名称（参数）；
如果没有定义构造函数，PHP会自动创建一个不带参数的默认构造函数

访问类中的成员属性和方法
$变量名 = new 类名称();
$变量名->成员属性= 值；
$变量名->成员方法；

$this  引用当前对象
::     没有任何实例的情况下访问类中的成员
parent关键字表示可以调用父类中的成员变量、常量和方法
self  可以调用当前类中的常量和静态成员
类名关键字可以调用本类中的常量和方法

- 构造函数和析构方法
在PHP5版本之前，构造方法的名称必须与类名相同
PHP5版本之后，构造方法的名称必须以两个下划线开头，“__construct”
function __construct($name,$gender){
		$this->name = $name;
		$this->gender = $gender;
}
析构方法  在对象被销毁的时候调用执行的，由于PHP具有自动垃圾回收机制，从而释放更多的内存，析构方法在垃圾回收程序执行前被调用的方法，是PHP编程中的可选内容。

- 访问方法
_get()   _set  类属性被访问和操作，访问方法都会被激活


- 类的继承
extends   单继承

- 高级特性
静态属性和方法
声明类属性或方法为static，就可以不实例化类而直接访问
静态属性不能通过一个类已实例化的对象来访问（但静态方法可以）
由于静态方法不需要通过对象即可调用，所以伪变量$this在静态方法中不可用
静态属性不可以由队形->操作符访问，
静态属性不需要实例化就可以直接使用，“类名：：静态属性名”
静态方法也不需要实例化即可直接使用  “类名：：静态方法”

final类和方法
如果父类的方法被声明为final，则子类无法覆盖该方法
如果一个类被声明为final，则不能被继承

- 抽象类和接口
抽象类
抽象类只能作为父类使用，用关键字abstract来声明，
抽象类和普通类的区别在于，抽象类的方法没有方法内容，而且至少包含一个抽象方法

```
<?php
	abstract class myObject{
		abstract function service($getName,$price,$num);
	}
	
	class myBook extends myObject{
		function service($getName,$price,$num){
			echo $getName."买了".$num."本书，总价".$price;
		}
		
	}
?>
```

接口
想实现多继承，就需要使用接口
interface声明，
接口中不能声明变量，只能使用关键字const声明为常量的成员属性，
接口中声明的方法必须是抽象方法，并且接口中所有的成员都必须具有public访问权限
implement 关键字实现接口
实现接口的类必须实现接口中声明的所有方法，除非这个类被声明为抽象类

```
interface Maxmin{
	public function getMax();
	public function getMin();
}

class man implement Maxmin{
	private $a=0;
	priavte $b=9;
	
	public function getMax(){
		return $this->b;
	}
	public function getMin(){
		return $this->a;
	}
}

$msm = new man();
echo msm->getMax();
```

面向对象的多态性（同一操作作用于不同的类的实例，将产生不同的执行结果）

通过继承实现多态
抽象类蔬菜    子类马铃薯   子类萝卜

通过接口实现多态
接口蔬菜     马铃薯类实现接口   萝卜类实现接口

- 异常（用于在指定的错误发生时改变脚本的正常流程）
异常触发时
当前代码状态被保存
代码执行被切换到预定义的异常处理器函数
根据情况，处理器也许会从保存代码的状态重新开始执行代码，终止脚本执行，或从代码中另外的位置继续执行脚本。

try代码块：使用异常的函数应该位于try块内，如果没有触发异常，则代码将照常继续执行，但是如果异常触发，会抛出一个异常。
throw代码块；这里规定如何触发异常，每一个throw必须对应至少一个catch
catch代码块:catch代码块或捕获异常，并创建一个包含异常信息的对象

```
<?php
	function checkNum($num){
		if($num>1){
		throw new Exception("数值必须小于或等于1");
		}
		return true;
	}
	try{
	 checkNum(3);
	 echo "没有异常";
	}
	catch(Exception $e){
		echo '异常信息：'.$e->Message();
	}
?>
```



*2019-03-07 10:14:55 星期四
萧逸小杨*