---
title: JavaWeb（jsp）
date: 2020-01-08 16:27:22
tags: [JavaWeb]
categories: [学习笔记]
---

 JSP是一种运行在服务器端的脚本语言

<!--more-->

**JSP程序运行机制**

当客户端请求JSP文件时，Web服务器执行该JSP文件，然后以HTML的格式返回给客户（浏览器显示），即，JSP程序的执行是由Web服务器（常用Tomcat服务器）来完成的

客户端请求JSP页面，服务器端的JSP引擎解释执行JSP代码，将JSP页面代码转换为servlet（.java）文件，然后借助JDK编译生成字节码文件（.class），然后生成servlet实例返回给客户端。

当Web客户端发送来一个请求时，web服务器首先判断是否为jsp请求
如果只是一般的html请求，则直接将html页面代码传给web浏览器
如果请求的是jsp页面，则由JSP引擎检查该Jsp页面

若该JSP页面不是第一次被请求，或者被修改，则JSP引擎将JSP页面代码转换为Servlet代码，然后进行编译生成字节码文件，再执行并将结果返回web浏览器

若该JSP页面时第一次被请求，则JSP直接调用Java虚拟机执行已经编译过的字节码文件，然后根据字节码的执行结果，生成对应的纯HTML的字符串给web浏览器

**JSP基本元素**

<%%>标签之内的称为JSP元素的内容

JSP声明：
在JSP页面中可以声明变量和方法，声明后的变量和方法可以在本JSP页面的任何位置使用，并在JSP页面初始化时被初始化。

```
    语法格式： <%! 声明变量、方法和类 %>
```

JSP表达式：
JSP的表达式是由变量、常量组成的算式，它将JSP生成的数值转换成字符串嵌入HTML页面，并直接输出（显示）其值。

```
语法格式：<%=表达式%>    
```

注意：表达式后没有“；”

JSP代码块：
JSP代码段可以包含任意合法的Java语句。

```
   语法格式：<% 符合java语法的代码块 %>
```


JSP注释：

```
<%-- 要添加的文本注释 --%>
```
功能：在JSP程序中，当在发布网页时完全被忽略，不以HTML格式发给客户（浏览器查看源码看不到）

HTML注释的语法格式：

 ```
 <!--要添加的文本注释--> （浏览器查看源码能看到）
 ```

Java注释语法格式（浏览器查看源码看不到） ：

```
 <%//要添加的文本注释  %>
或   <%/*要添加的文本注释*/%>
```


**JSP指令元素**

taglib指令：引用自定义的标签或第三方标签库

page指令：定义整个页面的全局属性。 

```
<%@page language="java" %>
```

1.在一个页面中可以使用多个%@page%指令，分别描述不同的属性
2.每个属性只能用一次，但是import指令可以被多次使用
3.page指令区分大小写
4.所有属性设置都是可选，只有language属性采用默认值java

include指令：用于包含一个文本或代码的文件。

```
<%@ include file="filename"%>
```

其中：include指令只有一个file属性，filename指被包含的文件的名称（相对路径），被插入的文件必须与当前JSP页面在同一Web服务目录下

（**静态包含**：被插入文件内容代替该指令标签，与当前JSP页面合并成一个JSP页面，两个文件在部署时，经编译合成一个.class文件）



**JSP动作元素**

```
<jsp:include>
```
在页面得到请求时动态包含一个文件。


```
<jsp:include page="文件的名字"/>
```

**动态包含**：当前JSP页面动态包含一个文件，即将当前JSP页面、被包含的文件各自独立编译为字节码文件。当执行到该动作标签处，才加载执行被包含文件的字节码。（动态包含可以在两文件之间传递参数）

```
<jsp:forward>
```

引导请求进入新的页面（转向到新页面）(*使用forward动作跳转到下一个页面后，浏览器地址栏的地址不变，还是前一个页面的地址*)


```
<jsp:param name="user"  value="">
```

传递变量值

request.getParameter("user)

需作为jsp:include或jsp:forward的子标记一起使用

**参数传递原理**：使用param标记传递参数，实际上是将数据信息，以name属性值为变量名，将该变量及其值保存到请求对象request中，在两一个文件中，再从request对象中获取数据信息

```
<js:include  page="文件的名字"> 
            <jsp:param name="变量名字1" value="变量值1" />
            ……
     </jsp:include>

```


**JSP内置对象**

**request**（当客户端通过HTTP协议请求一个JSP页面时，JSP容器（Tomcat）会自动创建request对象并将请求信息包装到request对象中，当JSP容器处理完请求后，request对象就会销毁。）



传参形式

1.使用JSP的forward 或include动作，利用传参数子动作<jsp:param>实现传递参数。

2.在JSP页面或HTML页面中，利用表单传递参数

3.追加在网址后的参数传递或追加在超链接后面的参数。(http://127.0.0.1:8080//infoReceive.jsp?rdName=abcdef&phName=123456789 )

4.超链接    <a href="infoReceive.jsp?rdName=abcdef&phName=123456789">传参</a>

特殊获取：采用getParameterNames()方法返回客户端传送给服务器端的所有参数名，结果集是一个Enumeration类的实例

新属性的设置与获取

request.setAttribute("key",Object);、
request.getAttribute(String name);

input.jsp

```
<form action="ch03_9_sum.jsp" method=post>
数据1: <input type="text" name="shuju1" ><br>
数据2: <input type="text" name="shuju2" ><br>
<input type="submit" value="提交" >
</form>
```

sum.jsp

```
<%  String str1=request.getParameter("shuju1");
String str2=request.getParameter("shuju2");
double s1=Double.parseDouble(str1);
double s2=Double.parseDouble(str2);
double s3=s1+s2;
request.setAttribute("st1",s1);
request.setAttribute("st2",s2);
request.setAttribute("st3",s3);
%>
<jsp:forward page="ch03_9_output.jsp"></jsp:forward>
```

output.jsp

```
利用getAttribute方法获取利用setAttribute方法保存的值，并显示！<br>
<% Double  a1=(Double)request.getAttribute("st1");
Double  a2=(Double)request.getAttribute("st2");
Double  a3=(Double)request.getAttribute("st3");
%>
<%=a1%>+<%=a2%>=<%=a3%><br>
利用getParameter方法获取获取请求参数，并显示（jsp：forward带着request中的信息跳转）！<br>
<%   String  s1=request.getParameter("shuju1");
String  s2=request.getParameter("shuju2");
%>       
```

**reponse**对象（服务器对客户端请求的响应，由服务器向客户端输出信息）

当服务器向客户端传送数据时，JSP容器会自动创建response对象并请信息封装到response对象中，当JSP容器处理完请求后，response对象会被销毁

response.sendRedirect("login_ok.jsp");

**重定向与转发**
（1）使用<jsp:forward>只能在本网站内跳转，而使用response.sendRedirect跳转到任何一个地址的页面；
（2）<jsp:forward>带着request中的信息跳转；sendRedirect不带request信息跳转。


**session**对象：会话(session)的含义：用户在浏览某个网站时，从进入网站到浏览器关闭所经过的这段时间称为一次会话。

一个客户对同一网站不同网页的访问属于**同一会话**。
当客户重新打开浏览器建立到该网站的连接时，JSP引擎为该客户再创建一个新的session对象，属于一次**新的会话**

```c++
session的创建时间是:<%=new Date(session.getCreationTime())%> <br>
session的Id号:<%=session.getId()%><br>
客户最近一次访问时间是:
<%=new java.sql. Time(session.getLastAccessedTime())%> <br>
两次请求间隔多长时间session将被取消(s):
<%=session.getMaxInactiveInterval()%> <br>
是否是新创建的session:<%=session.isNew()?"是":"否"%>
```



**application**对象（在服务器启动时对每个Web程序都自动创建一个application对象，只要不关闭服务器，application对象将一直存在，所有访问同一工程的用户可以共享application对象）

1.session对象与用户会话有关，不同用户的session是完全不同的对象，在session中设置的属性只在当前用户的会话范围内有效

2.所有访问统一网站的用户，都有同一个application对象服务器启动后就产生了这个application对象


```c++
<%!  Integer yourNumber=new Integer(0);%>
<%   if (session.isNew()){  //如果是一个新的会话
Integer number = (Integer) application.getAttribute("Count");
if (number == null) //如果是第1个访问本站
{ number = new Integer(1); } 
else 
{ number = new Integer(number.intValue() + 1); }
application.setAttribute("Count", number);
yourNumber = (Integer) application.getAttribute("Count");
}
%>
欢迎访问本站，您是第  <%=yourNumber%>个访问用户。
```

**out**对象

out对象主要用来向客户端输出各种格式的数据，并且管理服务器上的输出缓冲区。

HTML标记可以作为out输出的内容。我们之前在查看由JSP转换后的.java文件中可以看到，Servlet正是通过out对象，将JSP中所有的模板元素（HTML标签之类）和内容显示都以out对象的方法将其写入response对象中的。

写入源文件带换行，但是在网页解释html代码时忽略换行







------------

2019-06-23 09:46:10 星期日