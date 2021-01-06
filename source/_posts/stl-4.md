---
title: STL---->string
date: 2019-12-30 20:28:53
tags: [STL]
categories: [学习笔记]
---

 String 表示可变长字符序列，

<!--more-->



#### string

```
头文件<string>

c语言支持string类的头文件<string.h>
```


#### 定义

| 构造函数     | 函数原型                                             | 含义                       |
| ------------ | ---------------------------------------------------- | -------------------------- |
| 无参构造函数 | string()                                             | 空的字符串                 |
| 带参构造函数 | string(size_type length,char ch)                     | length个ch字符组成         |
|              | string(const char星 str)                             | 传一个字符串               |
|              | string(const char星 str,size_type length)            | 字符串的前length个字符     |
|              | string(string &str,size_type index,size_type length) | str中index开始length个字符 |
|              | string(const string &str)                            | 拷贝构造                   |


#### 属性

容量

| 容量       |                                                              |
| ---------- | ------------------------------------------------------------ |
| capacity() | 不重新分配内存空间的话，它可以容纳多少元素                   |
| 大小       | 在VS中，如果string的大小小于15，则申请15个容量，以后容量不够就每多申请16个；-------------------------------在VC6.0中，如果string的大小小于31个，则申请31个空间，以后每次增加32个 |
| 策略       | 当容器不得不获取新的内存空间时，vector和string的实现通常会分配比新的空间需求更大的内存空间，容器预留这些空间作为备用，可用来保存更多的新元素，这样，就不需要每次添加新元素都重新分配元素空间了 |
| 优点       | 如果每次添加一个元素，每次都重新分配内存空间的话，效率太低，所以一次申请15个内存，当不够的时候才每次新增16个 |

length()

|           |                                                              |
| --------- | ------------------------------------------------------------ |
| length()  | 字符串长度                                                   |
| size()    | 字符个数                                                     |
| resize()  | 重新设置字符个数，如果设置的小，截断字符串，size()变小，capacity不变；如果设置的大，size()变大，capacity变大，策略还是区间式增长 |
| reserve() | 预留空间，如果设置的小，则size和capacity都不变，如果设置的大，则size不变，capacity按照区间式增长 |


输出

|              |            |                                      |
| ------------ | ---------- | ------------------------------------ |
| 输出全部     | <<         | string 类重载了 << 运算符            |
| 输出全部     | c_str()    | 返回字符串首地址，类型是const char星 |
| 输出单个字符 | []下标运算 | string类重载下标运算符号             |
| 输出单个字符 | at()       | 返回对应字符的引用                   |

修改

| insert       |                                                              |
| ------------ | ------------------------------------------------------------ |
| 修改指定元素 | []下标运算                                                   |
| 修改指定元素 | at()赋值                                                     |
| 中间插入     | basic_string& insert( size_type index, const basic_string& str ); |
| 中间插入     | basic_string& insert( size_type index, const CharT* s );     |
| 中间插入     | basic_string& insert( size_type index, size_type count, CharT ch ); |
| 中间插入     | basic_string& insert( size_type index, const CharT* s, size_type count ); |
| 中间插入     | basic_string& insert( size_type index, const basic_string& str,size_type index_str, size_type count ); |
| 尾巴插入     | +=                                                           |
| 尾巴插入     | +=                                                           |

| append                                                       |      |
| ------------------------------------------------------------ | ---- |
| basic_string &append( const basic_string &str );             |      |
| basic_string &append( const char *str );                     |      |
| basic_string &append( const basic_string &str, size_type index, size_type len ); |      |
| basic_string &append( const char *str, size_type num );      |      |
| basic_string &append( size_type num, char ch );              |      |
| basic_string &append( input_iterator start, input_iterator end ); |      |



| 赋值                                                         |      |
| ------------------------------------------------------------ | ---- |
| =                                                            | 赋值 |
| >>                                                           | 输入 |
| assign                                                       |      |
| string& assign (const string& str);                          |      |
| string& assign (const string& str, size_t subpos, size_t sublen); |      |
| string& assign (const char* s);                              |      |
| string& assign (const char* s, size_t n);                    |      |
| string& assign (size_t n, char c);                           |      |


|                                                              |          |
| ------------------------------------------------------------ | -------- |
| 比较                                                         |          |
| int compare (const string& str) const;                       |          |
| int compare (size_t pos, size_t len, const string& str) const; |          |
| int compare (size_t pos, size_t len, const string& str,size_t subpos, size_t sublen) const; |          |
| int compare (const char* s) const;                           |          |
| int compare (size_t pos, size_t len, const char* s) const;   |          |
| int compare (size_t pos, size_t len, const char* s, size_t n) const; |          |
| compare                                                      | 大于返回 |

|                                                          |                                        |
| -------------------------------------------------------- | -------------------------------------- |
| 复制                                                     |                                        |
| size_t copy (char* s, size_t len, size_t pos = 0) const; |                                        |
| copy                                                     | 将对象中的某一段复制进另外一个字符数组 |

|                                                          |      |
| -------------------------------------------------------- | ---- |
| 查找子串                                                 |      |
| size_t find (const string& str, size_t pos = 0) const;   |      |
| size_t find (const char* s, size_t pos = 0) const;       |      |
| size_t find (const char* s, size_t pos, size_t n) const; |      |
| size_t find (char c, size_t pos = 0) const;              |      |

|                                                          |      |
| -------------------------------------------------------- | ---- |
| 返回子串                                                 |      |
| string substr (size_t pos = 0, size_t len = npos) const; |      |

|                          |      |
| ------------------------ | ---- |
| 交换                     |      |
| void swap (string& str); |      |


### 迭代器


iterator

**迭代器失效**

1.内存空间被重新分配的失效情况

如果添加的元素超过了容器的容量，则容器会新申请更大容量的内存空间，再将之前旧容器的元素拷贝进来，再添加新元素，此时，迭代器全部失效

2.内存空间未被重新分配

如果添加的元素没有超过元素容量，则直接添加到了现在容器的末尾，则其尾后迭代器会失效；如果是插入操作，插入位置后的迭代器都失效；如果是删除，删除位置后的迭代器都失效


### 算法




2019-07-12 21:00:00 星期五