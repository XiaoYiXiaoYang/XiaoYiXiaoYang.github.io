---
title: MyBatis---->Day1
date: 2020-01-08 16:35:04
tags: [JavaWeb]
categories: [学习笔记]
---

 MyBatis 是支持普通 SQL 查询,存储过程和高级映射的优秀持久层框架

<!--more-->

- MyBatis是一个优秀的持久层框架，它对jdbc的操作数据库的过程进行封装，使开发者只需要关注 SQL 本身，而不需要花费精力去处理例如注册驱动、创建connection、创建statement、手动设置参数、结果集检索等jdbc繁杂的过程代码。

- Mybatis通过xml或注解的方式将要执行的各种statement（statement、preparedStatemnt、CallableStatement）配置起来，并通过java对象和statement中的sql进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射成java对象并返回

1.XML配置文件

新建项目

- 新建文件lib，导入各种包

包括数据库驱动jar包，mybatis的jar包和一些依赖包

- 选中所有jar包，右击build path，生成path

- 为项目新增mybatis-config.xml配置文件

其中

```
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/students"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
      </dataSource>
    </environment>
  </environments>

 <mappers>
	    <mapper resource="stu/student.xml"/>
 </mappers> 
```

其中配置了数据库连接信息，

mapper标签指定了映射的文件

- 新建包stu

在stu下新建Student.java 和student.xml

Student.java是封装的JavaBean，需要和数据库对应

student.xml配置文件写出了数据库操作

```
<mapper namespace="a">
  <select id="selectStu" parameterType="int" resultType="stu.Student">
    select * from stu_info where stu_no = #{stu_no}
  </select>
</mapper>
```

mapper指定命名空间为“a”
查询操作的id为“selectStu”
参数类型为int
返回值类型为Student类
数据库查询语句为select * from stu_info where stu_no = #{stu_no}

这里#{stu_no}为传递的参数

- 新建Test.java 测试

```
//加载核心文件
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);

//
SqlSessionFactory SqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

SqlSession session = SqlSessionFactory.openSession();
System.out.println(session);

//查询
Student stu=session.selectOne("a.selectStu",1); 	System.out.println(stu.getStu_no()+"========"+stu.getStu_age());*/
```



2.接口+XML配置文件(推荐)


3.接口+注解方式





这块学的一知半解，对Java的深入就先到这里





2019-07-16 11:35:17 星期二