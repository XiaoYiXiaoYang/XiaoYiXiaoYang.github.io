---
title: go-codereview-1
date: 2021-06-23 22:46:08
tags: [GoLang]
categories: [学习笔记]
---

<center>
codereview
</center>

<!--more-->

1.golang codereview注意事项

参考链接：
https://github.com/golang/go/wiki/CodeReviewComments

https://golang.google.cn/doc/effective_go

## effective go

**go fmt**

go fmt程序运行在包级别，并且以标准样式的缩进和垂直方式对齐

缩进：使用制表符缩进
行长：没有限制
括弧：比c和java用的少

**注释**

go 提供 /**/块注释和 // 行注释

块注释出现在包注释处。出现在顶级声明之前的注释，中间没有换行符，与声明一起被提取出来作为项目的解释性文本。

每个包都应该有一个包注释，包注释只需要出现在一个文件中，任何一个都可以。包注释应介绍包并提供与整个包相关的信息。


## Go Code Review Comments

**gofmt**
格式化代码

注释句子
注释应以所描述事物的名称开头并以句点结束。

**context**
context.Context 类型的值携带安全凭证、跟踪信息、截止日期和跨 API 和进程边界的取消信号。Go 程序在从传入的 RPC 和 HTTP 请求到传出的请求的整个函数调用链中显式地传递上下文。

**copy**
拷贝结构时需要小心内部数据结构含有切片时，可能修改底层数组。

**随机数**
不要使用math/rand，应该使用crypto/rand

**声明空切片**

var t []string

而不是
t := []string{}
第一种声明了nil切片值，也就是长度为0，容量为0
第二种声明了空切片，长度为0

但是有时候首选第二种，比如在编码JSON对象时，nil切片编码为null对象，而[]string{}编码为空JSON数组

在设计接口时，避免区分零切片、零长度切片。

**文档注释**
包级别，导出的变量、函数声明等，应该有注释

不要使用panic
不要使用panic到常规的错误处理，使用error和多个返回值。

**Error strings**
不应该大写和以标点符号结尾。

**Goroutine 生命周期**
待补充

**处理Errors**
不要使用_处理错误，如果函数返回Error，需要处理Error

**import**
避免重命名导入，除非避免命名冲突。如果发生冲突，最好重命名本地项目的导入
导入按分组组织，他们之间空行，标准库，第三方库，本地的其他库

**导入Blank**
`import _ "pkg"`只能在main包使用

**导入Dot**
在不能循环依赖导入时很有用

package foo_test

```
import (
	"bar/testutil" // also imports "foo"
	. "foo"
)
```

**In-Band Errors**

和c类似，返回`-1`或者`null`表示错误，或者没结果

```
// 如果找不到返回""
func  Lookup ( key  string ) string
```

Go 对多个返回值的支持提供了更好的解决方案。

```
//如果没找到 ok=false
func  Lookup ( key  string ) ( value  string , ok  bool )
value, ok := Lookup(key)
if !ok {
	return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

返回值如 nil、””、0 和 -1 当它们是函数的有效结果时是可以的，也就是说，当调用者不需要以与其他值不同的方式处理它们时。

一些标准库函数，如包“strings”中的那些，返回带内错误值。这极大地简化了字符串操作代码，代价是需要程序员更加努力。一般来说，Go 代码应该为错误返回额外的值。

**缩进Error**
名称中的首字母缩写词或首字母缩略词（例如“URL”或“NATO”）具有一致的大小写。例如，“URL”应该显示为“URL”或“url”

**接口**

不要在使用之前定义接口：如果没有实际的使用示例，就很难判断一个接口是否是必要的

**Line Length**
Go 代码中没有严格的行长限制，但要避免令人不舒服的长行。
