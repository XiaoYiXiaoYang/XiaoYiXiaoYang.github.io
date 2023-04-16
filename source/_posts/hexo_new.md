---
title: Hello, Hexo
top: true
cover: false
toc: true
img: /medias/files/hexo.png
mathjax: true
date: 2020-01-14 15:27:31
password:
summary: Hexo是一个快速, 简洁且高效的博客框架. 让上百个页面在几秒内瞬间完成渲染. Hexo支持Github Flavored Markdown的所有功能, 甚至可以整合Octopress的大多数插件. 并自己也拥有强大的插件系统.
tags:
- Hexo
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
1.站点的_config.yml的 post_asset_folder: false改为true
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

| 配置选项          | 默认值                     | 描述                                                                                                   |
| ------------- | ----------------------- | ---------------------------------------------------------------------------------------------------- |
| title         | markdown文件标题            | 文章标题，强烈建议填写此选项                                                                                       |
| date          | 文件创建时的日期时间              |                                                                                                      |
| author        | 根 _config.yml 中的 author |                                                                                                      |
| img           | featureImages 中的某个值     | 文章特征图，推荐使用图床                                                                                         |
| top           | true                    | 推荐文章(文章是否置顶)，如果top为true，则会作为首页推荐文章                                                                   |
| cover         | false                   | 表示该文章是否需要加入到首页轮播封面                                                                                   |
| coverImg      | 无                       | 表示该文章在首页轮播封面需要显示的图片路径，如果没有，则默认使用文章的特色图片                                                              |
| password      | 无                       | 文章阅读密码，如果要对文章设置阅读验证密码的话，就可以设置                                                                        |
| toc           | true                    | 是否开启 TOC，可以针对某篇文章单独关闭 TOC 的功能。前提是在主题的 config.yml 中激活了 toc 选项                                         |
| mathjax       | false                   | 是否开启数学公式支持 ，本文章是否开启 mathjax，且需要在主题的 _config.yml 文件中也需要开启才行                                           |
| summary       | 无                       | 文章摘要，自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要                                          |
| categories    | 无                       | 文章分类，本主题的分类表示宏观上大的分类，只建议一篇文章一个分类                                                                     |
| tags          | 无                       | 文章标签，一篇文章可以多个标签                                                                                      |
| reprintPolicy | cc_by                   | 文章转载规则， 可以是 cc_by, cc_by_nd, cc_by_sa, cc_by_nc, cc_by_nc_nd, cc_by_nc_sa, cc0, noreprint 或 pay 中的一个 |

## 2023.04.16

文章插图问题：当我想在_post目录下创建images文件夹，存放图片

```
_posts/images/$filename/$imagename
```

然后在md中通过相对路径引用文件

```
![](../images/$filename/$imagename
```



但是由于hexo-image插件不能正确的将文件路径转换，导致转换后的路径和要引用的文件的路径不一致，也就导致了文章中图片不显示。

```
hexo-asset-image 已经过期
替代品：
hexo-asset-img
hexo-asset-link
```

注意：上述插件都要求将config.yaml中的post_asset_folder选项置为true
