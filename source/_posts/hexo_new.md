---
title: Hello, Hexo
top: true
cover: true
toc: true
mathjax: true
date: 2020-01-14 15:27:31
password:
summary: Hexo是一个快速, 简洁且高效的博客框架. 让上百个页面在几秒内瞬间完成渲染. Hexo支持Github Flavored Markdown的所有功能, 甚至可以整合Octopress的大多数插件. 并自己也拥有强大的插件系统.
tags:
- 博客
categories:
- 随笔
---

> Hexo是本次新挖掘出来的搭建网站的快速工具，容易上手，且样式好看。


### 起源

一直想要一个永久的域名，但是在学生时代经费紧张，就只能先这么凑合了。偶然有一次看到了Hexo + Github搭建个人网站的例子，Hexo是一款强大的产品，可以快速的生成页面。我也有尝试另外一款是wordpress，但是我本地测试后还有很多不易用的感受，而且我也不是专业的前端程序员，而Hexo可以根据网上的教程快速的搭建成自己舒服的样子，所以就被我pick。今天正式启用此站点，Hello Hexo！

## 2020.6.26

一次偶然的机会看到百度云可以1元得到一个域名和一个虚拟主机资源，所以产生了`www.yteng3456.xyz`，后来我也在这个域名上部署了php搭建的个人页面以及在复习专业课时总结的一些文章，但是后面时间到了域名失效了，如果要用还得重新备案；虚拟主机的资源也失效了，为了折腾，就把写的文章搬到了Hexo+Github去，就当是体验了一把个人网站上云。



## 2021.06.30

hexo 文章插图技巧
1.站点的_config.yml 的post_asset_folder: false改为true
2.在新建文章的时候`hexo new hello`就会在文章下生成同名文件夹，在文件夹中放图片，在文章中引用即可。

```
![](1.png)
```

hexo源码等保留到了github，换了电脑或者笔记本重装系统后需要重新部署环境
- 安装git，配置git账号信息，ssh key
- 安装nodejs
- 安装hexo `npm install -g hexo-cli `
- 安装git部署插件 `npm install hexo-deployer-git --save`
- 安装图片路径转换插件 `npm install https://github.com/CodeFalling/hexo-asset-image --save`

`hexo d`失败，报错：`typeError [ERR_INVALID_ARG_TYPE]: The "mode" argument must be integer. Received an instance of Object`
原因：nodejs版本过高，和hexo版本不匹配
解决办法：切换nodejs为低版本


## 2021.08.15

hexo文章编写技巧

`<!--more-->`该标签前面可以写文章摘要

`<center><center/>` 该标签可以把摘要居中

`<br/>`该标签可以插入一个换行

可以不用`![]`来插入图片，可以使用`<img src='' width='20' height='20>`来插入图片

## 2022.05.22

参考别人的主题进行了一次改版，参考网站：`https://godweiyang.com/`
Matery主题美化参考：`https://blog.csdn.net/kuashijidexibao/article/details/112971657`
为什么要改版？
因为之前的风格看的有些厌倦，不能让我很好的坚持写博客，不如换一个风格。
