---
title: JavaWeb（过滤器）
date: 2020-01-08 16:32:11
tags: [JavaWeb]
categories: [学习笔记]
---

 过滤器对用户的请求信息和响应信息进行过滤

<!--more-->

**过滤器技术**

过滤器是在服务器上运行的，在与过滤器相关联的Servlet或JSP等资源运行前，过滤器先执行

```
1.init(FilterConfig)
读取web.xml文件中Servlet过滤器的初始化参数

2.doFilter(ServletRequest,ServletResponse,FilterChain)
完成实际的过滤操作

3.destroy()
可以释放Servlet过滤器占用的资源。即使不添加代码，也要留着这个空方法
```


**设计过滤器**

创建和使用一个过滤器的基本步骤如下：

A．创建实现javax.servlet.Filter接口的类；

B．实现init方法，读取过滤器的初始化函数；

C．实现doFilter方法，完成对请求或过滤的响应；如果过滤器的验证没有通过，就可以不调用Filter链中的下一个Filter，可以用重定向方法定向到其他页面。

D．调用FilterChain接口对象的doFilter方法，向后续的过滤器传递请求或响应，若没有其他与Servlet或JSP页面关联的过滤器，就调用Servlet本身

E．web.xml中配置需要被过滤的URL。，使用filter元素和filtermapping元素


案例：基于过滤器的用户权限控制

```
public void doFilter(ServletRequest request,ServletResponse response,FilterChain filterchain){
	HttpSession session = request.getSession(true);
	if(session.getAttribute("u_name") == null){
		request.sendRedirect("login.jsp");
	}else{
		chain.doFilter(request,response);
	}
}
```



```
<filter>
<filter-name>FilterIP</filter-name>
</filter>
<filter-mapping>
	<filter-name>FilterIP</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>

```