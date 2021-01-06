---
title: JavaWeb（struts）
date: 2020-01-08 16:33:31
tags: [JavaWeb]
categories: [学习笔记]
---

 Struts是一个用于简化MVC框架开发的Web应用框架，具有组件模块化、灵活性和重用性等优点，使基于MVC模式的程序结构更加清晰，同时也简化了Web应用程序的开发

<!--more-->



**组成**

（1）模型组件：
模型组件是实现业务逻辑的模块，由JavaBean构成。

（2）视图组件：
视图组件主要有：HTML、JSP和Struts2标签等模板视图技术。

（3）控制器组件：
控制器组件主要由一个StrutsPrepareAndExecuteFilter核心控制器和业务控制器Action组成

**配置**


1. 包配置
Struts2把各种action分门别类地组织成不同的包，可以把包看成一个模块，一个struts.xml文件可以有一个或多个包。

```
<package name="包名称" namespace="/包的命名空间名" 
extends="struts-default">
在该包下的Action配置
</package> 
```

2. Action配置
Struts2中Action类的配置能够让Struts2知道Action的存在，并可以通过调用该Action来处理用户请求。Struts2使用包来组织和管理Action， <action >元素必须出现在<package >与</package>之间 。
Action的一般配置格式：

```
<action   name="名称" class="Action对应的类"  method="Action中某方法名" >
<result name="success">/page/hello.jsp</result>
</action>
```


3. result配置：

Action的result子元素用于配置Action跳转的目的地，一个action节点可能会有多个result子节点，多个result子节点使用name来区分。根据action的返回值定位到某个result的name，然后确定跳转地址。结果配置格式：

```
<result name="resultName" type="resultType">
跳转的目的地
</result>
```

result类型——type属性及其属性值

result的type属性值 struts-default包的result-types节点的name属性中定义，常见的type类型配置有：dispatcher、redirect、chain、redirectAction，默认的是JSP页面的转发。

（1）dispatcher： dispatcher是默认类型，表示转发到JSP页面，同Servlet中的转发。

（2）redirect：表示重定向，同Servlet中的重定向，重定向到JSP

（3）chain：表示转发到action。

（4）redirectAction：表明是重定向Action

4. 常量配置：

```
<constant name="struts.action.extension" value="do,,"></constant>
```

其中：
•name：表示常量名。
•value：表示常量值。


**Action实现类**

•  普通的Java类作为Action
一个普通的Java类可作为Action使用，但该类除所必需的属性外，通常包含一个execute()普通方法，该方法并没有任何参数，并返回字符串类型。


•  继承ActionSupport实现Action


•  对象属性驱动的Action


**Action访问Web对象**

直接访问Web对象——Servlet依赖容器方式。

```

HttpServletRequest request = ServletActionContext.getRequest();

HttpServletResponse response = ServletActionContext.getResponse();

ServletContext application = ServletActionContext.getServletContext();

PageContext pagecontext = ServletActionContext.getPageContext();

而HttpSession session的获取需要两步：
HttpServletRequest request = ServletActionContext.getRequest();
HttpSession session = request.getSession();

```

通过ActionContext访问——Map依赖容器方式。

通过IoC访问Servlet对象——Map IoC方式。

通过IoC访问Servlet对象——Servlet IoC方式


**多方法的Action**

为Action配置method属性

```
<package name="default" namespace="/" extends="struts-default"> 
<action name="add" class="Action.Ch11_3_Action" method="add">
<result name="show">/ch11_3_Show.jsp</result>
</action>
<action name="sub" class="Action.Ch11_3_Action" method="sub">
<result name="show">/ch11_3_Show.jsp</result>
</action>
<action name="mul" class="Action.Ch11_3_Action" method="mul">
<result name="show">/ch11_3_Show.jsp</result>
</action>
<action name="div" class="Action.Ch11_3_Action" method="div">
<result name="show">/ch11_3_Show.jsp</result>
</action>
</package>
```

动态方法调用

格式：
```
<form action="Action名字!方法名字">
```

```
<package name="default" namespace="/“   extends="struts-default"> 
<action name="FourOp" class="Action.Ch11_3_Action">
<result name="show">/ch11_3_Show.jsp</result> 
</action> 
</package>
```

使用通配符映射方式

```
<package name="default" namespace="/" extends="struts-default"> 
<action name="FourOp_*" class="Action.Ch11_3_Action"  method="{1}">
<result name="show">/ch11_3_Show.jsp</result> 
</action> 
</package>
```

**struts拦截器**


（1）自定义一个实现Interceptor接口的类（需要实现接口的三个方法）；或继承AbstractInterceptor（不需要实现没用的方法）；或继承MethodFilterIntercepter类完成对方法的拦截。

```
public class TestInterceptor implements Interceptor{
@Override
public void destroy() {
// TODO Auto-generated method stub
}
@Override
public void init() {
// TODO Auto-generated method stub
}
@Override
public String intercept(ActionInvocation arg0) throws Exception {
// TODO Auto-generated method stub
      arg0.invoke();   //此方法类似于FilterChain的doFilter()方法，如果不调用invoke()而直接返回，则后续的拦截器将不被执行
      return null;
}
   }
```

（2）在struts.xml中注册上一步中定义的拦截器。

（3）在需要使用的Action中引用上述定义的拦截器。

**拦截器栈**

拦截器的配置在struts.xml文件中完成，在 <interceptors>标签中，用<interceptor>标签配置拦截器，用<interceptor-stack>配置拦截器栈，格式如下：
```
<interceptors>
<interceptor name=“interceptorName" class=“interceptorClass">
</interceptor>
<interceptor name=“interceptorName2" class=“Class2“></interceptor>
<interceptor-stack name="myStack">
<interceptor-ref name="defaultStack"></interceptor-ref>
<interceptor-ref name=" interceptorName "></interceptor-ref>
</interceptor-stack>
</interceptors>
```


在action节点中，通过<interceptor-ref>元素引入拦截器或拦截器栈，格式如下：
```
<action name="public" class="interceptor.PublicAction">
<interceptor-ref name="replace" />  <!--使用自定义拦截器 -->
<interceptor-ref name=“defaultStack” />  <!--使用默认拦截器栈 -->
<result name="success">/success.jsp</result>
<result name="login">/success.jsp</result>
</action>
```



------------

2019-06-25 20:14:27 星期二