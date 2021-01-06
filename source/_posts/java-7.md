---
title: JavaWeb（JDBC）
date: 2020-01-08 16:29:49
tags: [JavaWeb]
categories: [学习笔记]
---

 数据库连接的两种方法，1.JDBC驱动程序直接连接数据库 2.连接池技术连接数据库

<!--more-->

**连接步骤**

一个Java程序要访问数据库，需通过以下几步来完成：
第一，注册数据库驱动；
第二，建立数据库连接；
第三，建立语句对象，通过该语句对象将SQL语句传送给数据库，进行数据库操作；
第四，处理数据库操作结果；
第五，释放资源。

**1.注册数据库驱动**

驱动程序接口Driver
(1)将驱动程序文件添加到应用项目
复制“mysql-connector-java-5.1.39-bin.jar”到Web应用程序的“WEB-INF\lib”目录下

(2)加载注册指定的数据库驱动程序

```
Class.forName(“com.mysql.jdbc.Driver”);
```

**2.建立数据库连接**

驱动程序管理器DriverManager

```
String url = "jdbc:mysql://localhost:3306/" + dbName + "?user="
+ userName + "&password=" + userPwd + "&useUnicode=true&characterEncoding=UTF-8";
java.sql.Connection conn = DriverManager.getConnection(url);
```


**3.建立语句对象**

数据库连接接口Connection
java.sql.Connection接口负责与特定数据库的连接，形成连接对象。

| 方法                                                        | 功能                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| Statement statement = conn.createStatement();               | 创建并返回一个Statement实例，通常在执行无参数的SQL语句时创建该实例 |
| PreparedStatement pstatement = conn.PreparedStatement(sql); | 创建并返回一个PreparedStatement实例，通常在执行包含参数的SQL语句时创建该实例，并对SQL语句进行了预编译处理 |

```
动态SQL设置参数
PreparedStatement 对象.setXXX(position,value);
```


**4.执行SQL**

执行静态SQL语句接口Statement

| 方法               | 功能                                                         |
| ------------------ | ------------------------------------------------------------ |
| executeQuery(sql)  | 执行指定的静态SELECT语句，并返回一个永远不能为null的ResultSet实例 |
| executeUpdate(sql) | 执行指定的静态INSERT、UPDATE或DELETE语句，并返回一个int型数值，为同步更新记录的条数 |

执行动态SQL语句接口PreparedStatement

| 方法            | 功能                                                         | 返回值                                      |
| --------------- | ------------------------------------------------------------ | ------------------------------------------- |
| executeQuery()  | 执行前面包含参数的动态SELECT语句，并返回一个永远不能为null的ResultSet实例 | 返回一个结果集                              |
| executeUpdate() | 执行前面包含参数的动态INSERT、UPDATE或DELETE语句，并返回一个int型数值，为同步更新记录的条数 | 返回一个整数，表示执行SQL语句影响的数据行数 |


5.**处理操作结果**

访问结果集接口ResultSet

| 方法                              | 功能                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| next()                            | 移动指针到下一行；指针最初位于第一行之前，第一次调用该方法将移动到第一行；如果存在下一行则返回true，否则返回false |
| getRow()                          | 查看当前行的索引编号；索引编号从1开始，如果位于有效记录行上则返回一个int型索引编号，否则返回0 |
| String getString(int ColumnIndex) | 返回指定字段的以Java的String类型表示的字段值                 |




6.**释放资源**

假设建立的对象依次为：连接对象为conn（Connection conn），操作对象为pstmt （PreparedStatement  pstmt），得到的查询结果集对象为rs（RrsultSet rs），则需要依次关闭的对象是：

```
rs.close();
stmt.close();
con.close();
```


**实例：学生信息管理**

```
增：String sql = "insert into stu_info(stu_no,stu_name,stu_sex,stu_age,score_java,score_cpp)values(?,?,?,?,?,?)";

删：String sql="delete from stu_info where stu_no=?";

改：String sql = "update stu_info set stu_name=?,stu_sex=?,stu_age=?,score_java=?,score_cpp=? where stu_no=?";

查： String sql ="select * from stu_info where stu_sex=? and stu_age>=? and stu_age<=? and score_java>=? and score_java<=? and score_cpp>=? and score_cpp <=? ";
```


**数据源与连接池技术**

1．在服务器上添加MySQL数据库驱动程序
将MySQL数据库的驱动程序复制到Tomcat安装路径下的lib文件夹中

2．数据源参数配置
在Web工程目录下的META-INF\context.xml文件中，若在Web工程目录下META-INF不存在context.xml文件，则需要自己建立该文件

```
<Context>
<Resource name="jdbc/mysql"
type="javax.sql.DataSource"
auth="Container"
driverClassName="com.mysql.jdbc.Driver"
url=“jdbc:mysql://localhost:3306/数据库名字?useUnicode=true&amp;characterEncoding=UTF-8"
username="用户名字"
password="用户密码"
maxActive="4" 
maxIdle="2" 
maxWait="6000" />
</Context>
```

3.使用
通过三个步骤来使用数据源对象：
（1）获得对数据源的引用：
InitialContext ctx =new InitialContext();
DataSource ds=(DataSource)ctx.lookup("java:comp/env/jdbc/mysql");
（2）获得数据库连接对象：
Connection con = ds.getConnection();
（3）返回数据库连接到连接池：
con.close();

------------


2019-06-23 12:21:27 星期日