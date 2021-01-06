---
title: JavaWeb（JavaBean）
date: 2020-01-08 16:28:28
tags: [JavaWeb]
categories: [学习笔记]
---

 JavaBean是一些封装了数据和操作的功能类

<!--more-->

**JavaBean设计**

1.是一个公共类(public)
2.有一个公共无参构造方法
3.所有属性定义为私有的
4.对每个属性有get、set方法
5.定义JavaBean时，在一个包下

**在JSP中使用JavaBean**

一种方法是脚本代码直接访问JavaBean
一种是通过动作标签访问JavaBean


1.声明JavaBean对象

```
<jsp:useBean id="对象名" class= "类名" scope= "有效范围"/>
```

功能：在指定的作用范围内，调用由class所指定类的无参构造方法创建对象实例。若该对象在该作用范围内已存在，则不生成新对象，而是直接使用。
使用说明：
（1）class属性：用来指定JavaBean的类名，注意，必须使用**全类名**。
（2）id属性：指定所要创建的对象名称。
（3）scope属性：指定所创建对象的作用范围，其取值有四个：page、request、session、application，*默认值是page*。分别表示页面、请求、会话、应用四种范围，

```
<jsp:useBean id="c" class="beans.Add" scope= "request"/>
```

2.使用动作标签

a、简单JavaBean属性设置
在获得Javabean实例后，就可以对其属性值进行重新设置，设置属性值的格式：

```
<jsp:setProperty name="beanname" property="propertyname" value="beanvalue"/>
```

其中：beanname代表JavaBean对象名，对应<jsp:useBean>标记的id属性；propertyname代表JavaBean的属性名；beanvalue是要设置的值。在设置值时，自动实现类型转换（将字符串自动转换为JavaBean中属性所声明的类型）。

```
<jsp:useBean id="c" class= "beans.Add" scope= "session"/>
<jsp:setProperty name="c" property="shuju1" value="10"/>
<jsp:setProperty name="c" property="shuju2" value="20"/>
对象c的属性shuju1赋值10，属性shuju2赋值20
```

b、将单个属性与输入参数*直接关联*

```
<jsp:setProperty name="beanname" property="propertyname"/>
```
功能：将参数名称为propertyname的值提交给同JavaBean属性名称同名的属性。并自动实现数据类型转换

```
<jsp:setProperty name="c" property="shuju1" />   //在表单提交页存在name为suhju1的输入域
<jsp:setProperty name="c" property="shuju2" />
```

c、将单个属性与输入参数*间接关联*

若JavaBean的属性与请求参数的名称不同，则可以通过JavaBean属性与请求参数之间的间接关联实现赋值

```
<jsp:setProperty name="beanname"
property="propertyname"   //JavaBean的数据成员名
param="paramname"/>   //表单的name项
```

功能：将请求参数名称为paramname的值给JavaBean的propertyname属性设置属性值。


d、将所有的属性与请求参数关联

将所有的属性与请求参数关联，实现自动赋值并自动转换数据类型（注意：**属性名和提交的参数名相同**才能自动赋值）。

```
<jsp:setProperty name="beanname" property="*"/>
```
功能：将提交页面中表单输入域所提供的输入值提交到JavaBean对象中相同名称的属性中。


**访问JavaBean属性**

在JSP页面显示JavaBean属性值，需要使用<jsp:getProperty>动作标签。

```
<jsp:getProperty name="beanname" property="propertyname"/>
```
功能：获取JavaBean对象指定属性的值，并显示在页面上。
说明：jsp:getProperty动作标签是通过JavaBean中的get方法获取对应属性的值。

**多个JSP页面共享JavaBean**

在JSP中，对于<jsp:useBean>动作标记可以使用scope属性来指定bean存储的位置（作用域），可以让多个JSP页面共享数据

1. page共享：
默认值，使用非共享(作用域为页面)的bean。

2. request共享：
共享作用域为请求的bean。处理当前请求的过程中，bean对象存储在request对象中，可以通过getAttribute访问到它。

3. session共享：
共享作用域为会话的bean。bean会被存储在与当前请求关联的session中，和普通的会话对象一样，可以使用getAttribute访问到它们。

4. application共享：
共享作用域为应用（即作用域为ServletContext)的bean。Bean将存储在application中，由同一Web应用中的所有JSP共享，可以使用getAttribute访问到它们


案例：数据库访问JavaBean设计

使用JavaBean技术将对数据库的有关操作封装成JavaBean类，从而简化JSP页面









------------

2019-06-23 16:00:15 星期日