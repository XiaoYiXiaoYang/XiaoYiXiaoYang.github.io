---
title: wx.uploadfile踩坑记
date: 2020-01-08 21:15:53
tags: [微信小程序]
categories: [技术分享]
---

 返回值是String，JSON解析？？？

<!--more-->

#### 第一个重点

wx.uploadfile返回值是String

应该是习惯了wx.request返回值是Object的原因，需要什么就res.data输出


首先是**小程序端玩单机**

```
var str = '{"name":"小明","age":"18"}';
console.log(str)
var json = JSON.parse(str);
console.log(json)
```

输出也很美好

```
{"name":"小明","age":"18"}
{name: "小明", age: "18"}
age:"18"
name:"小明"
__proto__:Object
```

很明显，解析正常啊。


**再配合服务器玩联机**

服务器接口函数（thinkphp）

```
 $str = '{"name":"小明","age":"18"}';
 return $str;
```

小程序端(success 回调函数)

```
console.log(res.data)
var json = JSON.parse(res.data);
console.log(json)
```

我看着也是没毛病啊，输出如下。。

```
 {"name":"小明","age":"18"}
 
thirdScriptError
Unexpected token ﻿ in JSON at position 0;at api uploadFile success callback function
SyntaxError: Unexpected token ﻿ in JSON at position 0
    at JSON.parse (<anonymous>)
```

这是啥，因为0位置有个.？


**改版**

服务器端不变，
小程序端

```
console.log(res.data)
var str =res.data;
console.log(str)
var json = JSON.parse(str);
console.log(json)
```

看着也没毛病啊，输出走你

```
{"name":"小明","age":"18"}
 {"name":"小明","age":"18"}
 thirdScriptError
Unexpected token ﻿ in JSON at position 0;at api uploadFile success callback function
SyntaxError: Unexpected token ﻿ in JSON at position 0
    at JSON.parse (<anonymous>)
```

对不起，我佛咯，

其次，还尝试了stringify，先将res.data转换为string，再parse为json，结果不变，所以也算是失败

###### 总结一下，以上都失败了呢...，功能方面，我直接接受字符串了，但是这块到底什么问题，暂时无结果。有大佬指点的话，本人不甚感激啊

------------

*2019-05-02 11:12:41 星期四
萧逸小杨*