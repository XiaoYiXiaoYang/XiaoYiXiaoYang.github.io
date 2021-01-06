---
title: markdown转HTML遇到的问题
date: 2020-01-08 21:19:51
tags: [PHP]
categories: [技术分享]
---

 markdown转html不能含有textarea元素？

<!--more-->

------------
### 分享一下开发过程遇到的坑
- **PHP文件上传问题**
我给每篇文章都设置了一个缩略图，但是要上传文件，代码参考了ThinkPHP5.0框架开发的代码，下面也把代码粘贴出来

```
$file = request()->file('pic');
if($file){
// 移动到框架应用根目录/public/uploads/ 目录下
 $info = $file->move(ROOT_PATH . 'public' . DS . '/static/uploads');
 if($info){
  //拼接文件路径
  //使用date()方法计算出八位时间上传到uploads的文件所在的文件名就是这八位时间
 //使用$info->getFilename()方法获取文件名
 $data['pic'] = date('Ymd').'/'.$info->getFilename();
 }else{
  // 上传失败返回错误信息
   return $this->error($file->getError());
  }
};
```

直接使用了PHP提供的move函数将图片文件上传，并且将图片的时间命名和文件名存储在数据库，方便在取出后加上前面都一样的路径即可读出。

------------

- **MarkDown编辑器嵌入问题**
这个问题很是麻烦，我首先开发的管理员界面，在管理员后台页面嵌入MarkDown编辑器，还是比较简单，照着百度的方法
1. 引入所需css jquery js文件
1. 引入html元素textarea文本框
1. 引入js代码，使用了editor.md编辑器
这部分虽然简单，但是在存储的形式依然采用了MarkDown语法

在前台部分读出文章内容还需要将MarkDown语法转换为HTML格式，又遇到一些小坑
这里我展示一下我学习到的方法
1. 同样需要先引入一堆js文件

```
<link rel="stylesheet" href="__ADMIN__/markdown/css/editormd.css" />
         <script src="__ADMIN__/markdown/lib/underscore.min.js"></script>
         <script src="__ADMIN__/markdown/lib/sequence-diagram.min.js"></script>
         <script src="__ADMIN__/markdown/lib/flowchart.min.js"></script>
         <script src="__ADMIN__/markdown/editormd.min.js"></script>
```

1. 引入textarea元素

```
<div id="layout"  class="editor">
这里不能有textarea代码段，很奇怪
  </div>
```
这个问题暂时得不到解决，假装那里有个textarea标签

- 然后引入js代码

```js
<script type="text/javascript">
          $(function() {
                var testEditor;
          testEditor = editormd.markdownToHTML("test-editormd", {
         // markdown:$scope.apidetails.content,
          htmlDecode      : "style,script,iframe",  // you can filter tags decode
          emoji           : true,
          taskList        : true,
          tex             : true,  // 默认不解析
          flowChart       : true,  // 默认不解析
          sequenceDiagram : true,  // 默认不解析
          });
          });
        </script>
```

这样读取的数据库markdown语法的文章就能以html格式展示在页面啦

------------

*今日分享就到这里* 
*2019-03-02 11:24:26 星期六* 
*萧逸小杨*