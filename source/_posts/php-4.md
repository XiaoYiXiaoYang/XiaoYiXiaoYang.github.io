---
title: PHP基础（get/post/cookie/session）
date: 2020-01-08 11:24:21
tags: [PHP]
categories: [学习笔记]
---

 分享get、post、cookie、session

<!--more-->



### POST提交
post方式提交的变量、不受特定的变量大小的限制，并且被传递的变量不会再浏览器地址栏里以URL的方式显示在浏览器地址栏里

$POST[] 读取传递的数据，以表单元素的名称属性为键名，以表单元素的输入数据或传递的数据为键值

向服务器传送数据

POST通过HTTP POST机制，将表单中各个字段与其内容放置在HTML HEADER中一起传送到ACTION属性所指的URL地址，用户看不到这个过程

对于POST方式，服务器端用Request.Form获取提交的数据

POST传送数据量较大，一般被默认不受限制，但理论上在IIS4中最大值为80kb，在IIS5中最大值为100kb，



### GET提交

GET方式提交的变量有大小显示，不能超过100个字符，他的变量名和与之对应的变量值都会以URL的方式显示在浏览器地址栏里

$GET[] 读取传递的数据，以表单元素的名称属性为键名，以表单元素的输入数据或传递的数据为键值

从服务器上获取数据

GET是把参数数据队列添加到提交表单的ACTION属性所指的URL中，值和表单内各个字段一一对应，在URL中可以看到

对于GET方式，服务器端用Request。QueryString获取变量的值

GET传送数据量较小，不能大于2kb



### COOKIE

由于HTTP Web是无状态协议，对于事务没有记忆能力，缺少状态意味着如果后续处理需要前面的信息，则它必须重传，所以COOKIE、SESSION产生了

COOKIE将数据存储在客户端，并显示永久的数据存储
SESSION将数据存储在服务器端，保证数据再程序的单词访问中有效

COOKIE是服务器留在用户计算机中的小文件，每当相同的计算机通过浏览器请求页面时，他会同时发送COOKIE。
COOKIE工作原理：当一个客户端浏览器连接到一个URL，它会首先扫描本地存储的COOKIE，如果发现其中有和此URL相关联的COOKIE，将会把它们返回给服务器

setcookie(名称，cookie值，到期日，路径，域名，secure);
$_COOKIE["名称"];

删除COOKIE
手动删除（清空浏览器COOKIE）
函数删除（setcookie("名称","",0)）  //到期日为0，则直接删除这个COOKIE



### SESSION(会话管理)

在PHP中，每一个sessiom都有一个ID,这个sesion就是一个由PHP随机生成的加密数字，这个session id 通过cookie存储在客户端浏览器中，或者直接通过URL传递到客户端，如果在某个URL后面看到一长串加密数字，这很有可能就是session id了

创建会话session_start()函数，

```
<?php
	session_start();
?>
```
上面的代码会向服务器注册用户的会话，以便可以开始保存用户信息，同时会为用户会话分配一个UID

注册会话变量

```
<?php
	session_start(); //启动session
	$_SESSION['name']=yteng;  //声明一个name变量，值为yteng
?>
```

使用会话变量
首先要判断会话变量是否存在一个会话ID，如果不存在，则需要创建一个，并且能够通过$_SESSION变量进行访问，如果已经存在，则将这个已经注册的会话变量载入以供用户使用


```
<?php
	if(!empty($_SESSION['name']))   //判断是否为空
	$value = $_SESSION['name']; 
?>
```

注销会话变量
unset函数

```
<?php
	unset($_SESSION['name']); 
?>
```



------------

*2019-03-16 15:25:21 星期六
萧逸小杨*