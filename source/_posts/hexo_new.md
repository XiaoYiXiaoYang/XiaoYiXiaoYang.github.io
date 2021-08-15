---
title: Hello, Hexo
date: 2019-12-01 11:13:12
tags: [生活]
categories: [生活]
---

<center>
Hexo是本次新挖掘出来的搭建网站的快速工具，容易上手，且样式好看。
</center>
<!--more-->


一直想要一个永久的域名，但是苦于经费紧张，就只能先这么凑合了。Hexo也是一款强大的产品，是我目前所知建立个人网站比较快的，还有一款是wordpress，但是我本地安装后样式感觉有待改进，而Hexo可以自定义很多样式，所以比较受欢迎吧。今天正式启用此站点，Hello Hexo！


之前的网站www.yteng3456.xyz已经到期了，解析失效，几天之后，虚拟主机也会失效。


## 2020.6.26 更新

www.yteng3456.xyz域名已经失效，如果要用还得重新备案；虚拟主机的资源也失效了，所幸自己把虚拟主机的资源的下载了，算是一次体验。


今天将hexo文章插图弄了一下，没有图片的文章感觉没有灵魂。
具体操作：
1.站点的_config.yml 的post_asset_folder: false改为true
2.在新建文章的时候`hexo new hello`就会在文章下生成同名文件夹，在文件夹中放图片，在文章中引用即可。

```
![](1.png)
```



## 2021.06.30

1.hexo源码等保留到了github，重装系统后需要重新部署
- 安装git，配置git账号信息，ssh key
- 安装nodejs
- 安装hexo `npm install -g hexo-cli `
- 安装git部署插件 `npm install hexo-deployer-git --save`
- 安装图片路径转换插件 `npm install https://github.com/CodeFalling/hexo-asset-image --save`

2.发现`hexo d`失败。
报错：typeError [ERR_INVALID_ARG_TYPE]: The "mode" argument must be integer. Received an instance of Object
原因：nodejs版本过高，和hexo版本不匹配
解决办法：切换nodejs为低版本；参考：https://www.jb51.net/article/202124.htm



## 2021.08.15

1.hexo文章编写小技巧

`<!--more-->`该标签前面可以写文章摘要

`<center><center/>` 该标签可以把摘要居中

`<br/>`该标签可以插入一个换行

- 理论上所有html标签都可以用



