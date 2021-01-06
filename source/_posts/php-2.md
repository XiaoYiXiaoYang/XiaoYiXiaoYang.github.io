---
title: PHP基础（函数、字符串、数组、时间）
date: 2020-01-08 11:21:41
tags: [PHP]
categories: [学习笔记]
---

 复习php基础第二块内容

<!--more-->



- 函数
function name_of_function(param1,param2,...){
	statement
}

向函数传递参数值   function total（$fee）
向函数传递参数引用  function total（&$fee）  当函数改变$fee变量值，则函数外的$fee值也发生改变
函数返回值   

对函数的引用  function &example($a=1)
(在定义函数和引用函数时，都必须使用“&”符号，表明返回的是一个引用)

包含文件require  include
当文件读取失败后require将产生一个致命错误，而include产生一个警告

- 流程控制
if
if else
if elseif else
switch

while
do...while
for
foreach（经常被用来遍历数组元素）
foreach(数组 as数组元素){
	对数组元素的操作命令；
}

```
$arr = array("one","two","three");
foreach($arr as $value){
	echo "数组值".$value."<br />"; 
}
```

break
continue

- 字符串
单引号
双引号 (会输出变量值)

连接符 "."   {}
$a = "杨先生";
$b="欢迎{$a}";

strlen()   (计算字符串长度)
str_word_count()   （字符串单词统计） 只对基于ASCII码的单词起作用，不对UTF8中文字符起作用

清理字符串中的空格
ltrim()  (从左面清除)
rtrim()   （从右面清除）
trim()    （从两边同时去除头部和尾部的空格）

切分与组合
$b = explode(' ',$a)   按照空格切分成$b数组
$c = impolde('=',$b)   把数组$b按照=为间隔组合成新的字符串

字符串子串的截取  substr(目标字符串，起始位置，截取长度)
字符串子串替换substr_replace(目标字符串，替换字符串，起始位置，替换长度)
字符串查找 strstr(目标字符串。需查找字符串)
大小写转换  string strtolower(string str)   转换为小写

- 正则表达式
方括号 [name] 指在目标字符串中寻找字母n，a，m，e
连字符 -    [a-z]表示匹配英文字母小写a到z
点好字符 .    是一个通配符，表示所有字符和数字
限定符 （+*? {n,m}）   
+表示其前面的字符至少有一个
*表示其前面的字符不止一个或零
？表示其前面的字符为一个或零
大括号{n,m} 表示前面的字符有n个或m个
行定位符 ^$
^表示在目标字符串开头出现
$表示在目标字符串结尾出现

排除字符 [^]
排除匹配字符在目标字符串中出现的可能

括号字符 ()
表示子串，所有对包含在子串内字符的操作，都是以子串为整体进行的

选择字符 |
表示或选择

转义字符 \  与反斜线\
如果要表示反斜杠本身则 为\\

- 数组
1.使用array()声明数组
2.直接通过为数组元素赋值声明数组

数组排序   sort()
ksort()   根据数组元素键值升序排序
asort()   根据数组元素值升序排序

字符串与数组转换  explode 和 impolde

向数组中添加和删除元素
添加  push pop shift unshift
删除  array_shift   array_pop

查询数组中指定元素 in_array()  array_key_exists() array_search() array_keys()  array_values()

统计数组元素个数 count()
删除数组中重复元素  array_unique()
调换数组中键值和元素值 array_flip()

- 时间和日期
UNIX时间戳
时间存储为一个整数，这个数的开始时间是GMT 格林尼治 1970年1月1日，也就是现在的时间是GMT1970.01.01零点整到现在的秒数

获取当前时间戳 time()
获取当前日期和时间date()
获取时间戳整数 getdate() 
检验日期有效性  checkdate()

这一块内容较多，使用的时候记得查阅资料





*2019-03-06 22:22:22 星期三
萧逸小杨*