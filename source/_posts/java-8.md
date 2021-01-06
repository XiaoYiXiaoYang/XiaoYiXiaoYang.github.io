---
title: JavaWeb（Servlet）
date: 2020-01-08 16:30:57
tags: [JavaWeb]
categories: [学习笔记]
---

 Servlet技术可以处理客户端传来的HTTP请求，并返回一个响应

<!--more-->

**设计Servlet**

1.实现Servlet接口
创建一个Servlet类，必须直接或间接实现javax.servlet.Servlet接口

2.继承GenericServlet
GenericServlet是Servlet接口的直接实现类

3.继承HttpServlet
HttpServlet类是javax.servlet.GenericServlet类的一个子类

**Servlet类基本结构**

```
public class 类名 extends HttpServlet{
	public void init(){}
	public void doGet(HttpServletRequest request,HttpServletResponse response){}
	public void doPost(HttpServletRequest request,HttpServletResponse response){}
	public void service(HttpServletRequest request,HttpServletResponse response){}
	public destroy(){} 
}
```

service方法:Servlet处理请求时自动执行service()方法，该方法根据请求的类型（get/post），调用doGet()或doPost()方法，因此，在建立Servlet时，一般只需重写doGet()和doPost()方法


**配置Servlet**

新建web工程时，自动在WebRoot/WEB-INF目录下创建配置文件web.xml

```
<servlet>
<description>This is the description of my J2EE component</description>
<display-name>This is the display name of my J2EE component</display-name>
<servlet-name>NewsServlet</servlet-name>               //Servlet名称
<servlet-class>servlets.NewsServlet</servlet-class>   //完整的类路径
</servlet>

<servlet-mapping>
<servlet-name>NewsServlet</servlet-name>			//Servlet名称，和上面名称要一致
<url-pattern>/NewsServlet</url-pattern>		//访问地址
</servlet-mapping>
```

Servlet常用对象及其方法

| JSP内置对象 | Servlet类          |
| ----------- | ------------------ |
| request     | HttpServletRequest |
| response    | HttpServletRsponse |
| session     | HttpSession        |
| application | ServletContext     |
| out         | PrintWriter        |



| 对应关系                                       | 获取对象方法                                               |
| ---------------------------------------------- | ---------------------------------------------------------- |
| 类HttpServletRequest的对象对应JSP的request对象 |                                                            |
| 类HttpServletResponse对象对应JSP的response对象 |                                                            |
| 类HttpSession的对象对应JSP的session对象        | HttpSession session =request.getSession()：获取session对象 |
| 类ServletContext的对象对应JSP的application对象 | ServletContext app= this.getServletContext()               |
| 类PrintWriter的对象对应JSP的out对象            | PrintWriter out=response.getWriter();                      |


**在Servlet中使用JavaBean**

在Servlet使用JavaBean有两种方式：
（1）在一个Servlet中单独使用JavaBean(普通类的使用方法)
一般需要完成的操作是：
在Servlet中实例化JavaBean。
通过实例化的对象调用JavaBean方法，完成业务处理，并获得处理结果。
将获得的结果交给Servlet继续处理。

（2）在Servlet与JSP之间或Servlet之间实现共享的JavaBean

**Jsp与Servlet的关联关系**

JSP可以调用Servlet，Servlet也可以调用JSP；同时，一个JSP可以调用另一个JSP，一个Servle也可以调用另一个Servlet

1. JSP页面调用Servlet

（1）通过表单提交调用Servlet

```
<form action="servlet访问地址">……</form>
```
这里的访问地址是在web.xml中配置的地址。

（2）通过超链接调用Servlet

```
<a href="servlet访问地址">提示信息</a>
或  <a href="servlet访问地址?要传递的参数">提示信息</a>
```

2. Servlet跳转到JSP页面

转向：是在一个Web工程内部，各组件之间的调用，在调用时，request对象中的信息不丢失，可在新组件中继续使用。

```
RequestDispatcher rd=request.getRequestDispatcher("jsp网页")
rd.forward(request,response);
```

类似于JSP动作<jsp:forward page=“url”>，只在网站内部跳转，携带request信息。

重定向：可以在一个Web工程内部，各组件之间实现调用，也可以直接跳转到其他Web工程的JSP页面。并且，在跳转到新组件后，重新创建request对象。

```
response.sendRedirect("JSP网页地址");
```

3. Servlet调用另一个Servlet

其调用格式同Servlet调用JSP（转发和重定向），只是将JSP网页地址更换为Servlet映射地址即可。

4. JSP跳转到另一个JSP

<jsp:forward page=“jsp页面”>
表单提交，action=“jsp页面”
超链接jsp页面
response.sendRedirect(“jsp页面”)


**Cookie管理**

Cookie对象是由服务器产生并保存到客户端(内存或文件中)的信息

Cookie不是JSP的内置对象，在页面中使用时需要通过其构造方法创建。


| 方法                                  |                                                              |
| ------------------------------------- | ------------------------------------------------------------ |
| Cookie(String name, String value)     | 构造函数，创建一个Cookie对象                                 |
| **void response.addCookie(Cookie c)** | 向客户机添加一个Cookie对象                                   |
| **Cookie[]   request.getCookies()**   | 用于取得所有Cookie对象的数据，存放到一个数组中（不能直接获取某一个单独的Cookie对象，只能一次性将全部Cookie对象拿到） |
| String getName()                      | 获取cookie对象的属性名                                       |
| String getValue()                     | 取得Cookie对象的属性对应的属性值                             |
| setMaxAge(int expiry)                 | 设置cookie有效时间,以秒为单位，**设置了有效时间的cookie将以文件形式保存在客户端，否则只是存储在浏览器的内存区域中** |


**案例：Cookie实现自动登录**

登录页面：输入账号密码

logincheck页面：接收用户输入并创建cookie保存用户输入的用户名密码，同时在session中保存用户名，转入主页面

主页面：需要登录才能访问，先检验用户是否已经登录，如果已经登录则显示欢迎消息，如果没登录，则启动自动登录，读取cookie信息，如果读取到用户名信息，则显示欢迎信息，否则转登录页面。

**常用开发模式**

单纯的JSP页面开发

JSP+JavaBean

JSP+Servlet

JSP+JavaBean+Servlet

DAO设计模式与数据库访问







------------

2019-06-23 17:46:40 星期日