---
title: JavaWeb（html+css+js）
date: 2020-01-08 16:25:59
tags: [JavaWeb]
categories: [学习笔记]
---

 基本网页编程技术

<!--more-->

**JavaWeb程序的开发与部署**

建立JavaWeb项目，编写代码
部署（将myeclipse下的项目文件夹移动到服务器TomCat的webapps目录下）
启动TomCat服务器
若需要部署到其他服务器还要生成并发布war文件

**Web应用的目录结构**

WEB-INF目录

WEB-INF目录是一个专用区域，该目录下的文件只供容器使用，Web容器要求在应用程序中必须有WEB-INF目录。

WEB-INF中包含：
WEB-INF/web.xml文件：配置信息文件。
一个classes目录：WEB-INF/classes目录，编译后的Java类文件。
一个lib目录：WEB-INF/lib目录，Java类库文件（*.jar）。

src：java源文件。

公开目录（WebRoot）：公开目录存放所有可被的访问的资源： .html、.jsp、.gif、.jpg、.css、.js等。

**Tomcat目录结构**

/bin 启动和关闭tomcat的命令文件

/lib web应用及tomcat都可以访问的Jar文件

/conf 配置文件

/logs 日志文件

/temp 临时文件

/webapps web应用都放在该目录下

**/work** Tomcat将JSP生成的Servlet源文件和字节码文件自动存放在这个目录下

导出，file-》export-》...

导入file-》import-》General-》existing projects-》copy into workspace

可以将打包后的web工程部署到另外一台计算机的Tomcat服务器下，放在Tomcat下的webapps目录下，在浏览器地址栏访问war工程，服务器会自动解压

**HTML、CSS、JS重点**

**HTML标签**

单标签   ```<br/>```

双标签 `<h1></h1>`

结构标签

```
<html></html>
<title></title>
<head></head>
<body></body>
```

注释标签 `<!--  注释内容 -->`

列表标签   

    有序列表 <ol> 
    无序列表 <ul>  

属性type指定列表项前的项目符号的样式，disc：实心圆点（默认），circle（空心圆点），square（实心方块）  若要不显示可设置type=none

超链接    `<a>`  

链接资源：本web应用使用相对路径跳转，其他web应用带协议跳转

资源定位：在网页顶部先定义一个位置：`<a  name =“top”></a>`

在网页底部回到这个位置   `<a  href=“#top”>回到顶部</a>`

target：设置链接打开方式，默认在当前页打开。
                     _blank:在一个新窗口打开
                     _self:在当前页打开           

图片标记 （重点）

```
<img src="url" height="" width =""  alt="">
```
其中：
        属性src：指定图像源的URL路径
               height：图片的高度；
               width：图片的宽度；
               alt：替代文本（鼠标移动到图片上显示的内容，当图片出错或路径不对，会直接显示一个×和alt中的内容

表格标签：

```
<table>     
<th>  表头单元格，包含表头信息，（会自动加粗居中）
<tr>  标准单元格，包含数据
<td>
```

表单标签

```
<form name="表单名" method="提交方法" action="处理程序">
</form>
```

input标记

```
<input>表示输入域，是个单标记，必须嵌套在表单中使用

<input name="输入域名称" type="输入域类型">

type主要的name属性和type属性必选 
```

下拉列表框

```
<select name="" size="" multiple>
	<option></option>
	<option></option>
</select>
```

多行文本框标签

```
textarea name="" rows="" 
				cols=""
				wrap="off | virtual |physical"

	
/textarea
    textarea  wrap=
    	wrap设置是否自动换行（
    	off：不自动换行； 
    	virtual： 将实现文本区内的自动换行，以改善对用户的显示，但在传输给服务器时，文本只在用户按下 Enter 键的地方进行换行，其他地方没有换行的效果；
    	physical： 将实现文本区内的自动换行，并以这种形式传送给服务器，所见即所得。）
```


定时刷新

```
meta：设置页面的一些相关内容
1s刷新一次：<meta http-equiv=“refresh” content=“1” />
```

**CSS样式**

```
行内式 <p style="">
内嵌式 <style type = "text/css">   </
链接式	<link href="" type="" rel="stylesheet">
导入式	<style type=""> @import url(*.css文件路径);
```

优先级（就近）

标记选择器 标签名{}
类别选择器 .类名{}
ID选择器   #ID名{}



**JavaScript技术**

**JS与HTML结合的两种方式**

第一种：使用script标签在html文件中写js代码

```
<script type="text/javascript">
</script>
```

第二种：使用script标签引入js文件

```
<script type="text/javascript" src="js文件">
</script>
```

数据类型：number   string   boolean   null

字符串与数字相加，是字符串链接，如果相减，字符串直接转换成数字再相减

```
<script type="text/javascript">
	var st1="12";
	var st2="34";
	alert(st1+st2);   //1234
	alert(st1-st2);	//-22
</script>
```

boolean类型可以进行运算，false---0，true---1，其他类型转为boolean型规则：非零非空---true，0或空---false

== 和===的区别，==是判断值是否相等，===不但判断值还判断数据类型

```
var a=5;
if(a==5){ //结果输出a=5
	alert("a=5");
}else{
	alert("other");
}
var b="5';
if(b==5){ //结果输出b=5
	alert("b=5");
}else{
	alert("other");
}

var c="5";
if(c==5){ //结果输出other
	alert("c=5");
}else{
	alert("other");
}

```


JS函数定义和调用

JS形参前不需要写数据类型或var

（匿名函数）：
var  变量名 = function(参数列表) {
	函数体；
	返回值根据需要可有可无；
}

```
var add=function(a,b){  //函数声明
	var c=a+b;
	alert(c);
}

add(2,3);  //函数调用
```
JS全局变量和局部变量

全局变量：在script标签里定义的变量是全局变量，这个变量可以在整个页面的js部分都可以使用，即使相同页面的不同script标签，（但是注意先定义后使用）

局部变量：方法内部定义的变量是局部变量，只能在方法内部使用

script标签的位置

1.一般放在head标签中
2.如果没有任何参数的影响，可以放在html页面的任何位置



**JS事件**

onBlur  元素或窗口本身失去焦点时触发
onChange  当表单元素获取焦点，且内容值发生改变时触发
onClick   单击鼠标左键时触发
onFocus   任何元素或窗口本身获得焦点时触发
onKeydown  键盘键被按下时触发，如果一直按着某键，则会不断触发



内置对象 String，Date，Array，Math
浏览器的文档对象：window ，navigator， screen，history，location，document

window对象

alert(message)   弹出警告对话框
confirm(meaasge)  显示一个确认对话框，点击“确认”返回true，否则返回false

location对象

网页之间跳转
window.location.href=""

history对象

go(index)  从浏览历史加载url，index为负数，表示当前地址之前的浏览记录，index正数表示当前地址之后的浏览记录
forward()   加载下一个，相当于history.go(1)
back()       加载上一个，相当于history.go(-1)

document对象

document每个HTML文档被加载后都会在内存中初始化一个document对象，该对象存放整个网页HTML内容
getElementById：通过id得到元素（标签）
*getElementsByName*：通过标签name属性获得标签     ，返回带有指定名称的对象的集合。
*getElementsByTagName*：通过标签名得到元素，返回带有指定标签名的对象的集合，返回元素的顺序是它们在文档中的顺序。



------------

2019-06-23 10:01:20 星期日