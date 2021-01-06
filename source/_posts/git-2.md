---
title: 使用Git将项目上传至gihub（方法2）
date: 2020-01-08 21:25:42
tags: [Git]
categories: [技术分享]
---

 再次分享建立本地仓库，上传一个Java学习中敲的小程序

<!--more-->

##### 上一篇实战上传十分顺利，有点膨胀，想试试百度的第二种上传方法，就是在本地建立仓库的方式，不用每次上传都建立仓库

1.创建一个本地的版本库（其实也就是一个文件夹）(F:\GitHub\repository)

2.右键repository文件夹，git bash here

3.通过命令git init把这个文件夹变成Git可管理的仓库

4.这时会发现repository里面多了个.git文件夹，它是Git用来跟踪和管理版本库的

5.把项目粘贴到这个本地Git仓库里面，然后通过git add .把项目添加到仓库

6.用git commit -m "java first"把项目提交到仓库

7.下面就到了连接远程仓库,由于本地Git仓库和Github仓库之间的传输是通过SSH加密的，所以连接时需要设置一下

8.创建SSH KEY。先看一下C盘用户目录下有没有.ssh目录，有的话看下里面有没有id_rsa和id_rsa.pub这两个文件，有就跳到下一步，没有就通过下面命令创建

```
$ ssh-keygen -t rsa -C "1409026014@qq.com"
```
注意命令 ssh-keygen之间每空格，否则报错Bad escape character 'ygen'.

如果命令写对了，一路回车就好

9.登录Github,找到右上角的图标，打开点进里面的Settings，再选中里面的SSH and GPG KEYS，点击右上角的New SSH key，然后Title里面随便填，再把刚才id_rsa.pub里面的内容复制到Title下面的Key内容框里面，最后点击Add SSH key，这样就完成了SSH Key的加密。

10.在Github上创建好Git仓库之后我们就可以和本地仓库进行关联了，根据创建好的Git仓库页面的提示，可以在本地repository仓库的命令行输入：
git remote add origin https://github.com/XiaoYiXiaoYang/JavaBase.git

origin后面加的是Github上创建好的仓库的地址。

11.关联好之后就可以把本地库的所有内容推送到远程仓库（也就是Github）上了
git push -u origin master
然后根据提示输入账号密码

（由于新建的远程仓库是空的，所以要加上-u这个参数，等远程仓库里面有了内容之后，下次再从本地库上传内容的时候只需下面这样就可以了：）

总结一下，下次用这个仓库上传项目步骤就会少一些了

1.把项目复制到这个文件夹里面，再通过git add .把项目添加到仓库；

2、再通过git commit -m "注释内容"把项目提交到仓库；

3.SSH密钥在Github上已经设置好了，新建一个远程仓库，通过git remote add origin https://github.com/XiaoYiXiaoYang/test.git将本地仓库和远程仓库进行关联；

4.最后通过git push -u origin master把本地仓库的项目推送到远程仓库（若新建远程仓库的时候自动创建了README文件会报错，解决办法https://blog.csdn.net/Lucky_LXG/article/details/77849212）。


Git版本管理器在以后会用的越来越多，这些常规的操作也应该烂熟于心

Git常用操作命令
https://blog.csdn.net/Lucky_LXG/article/details/77849212

------------
*2019-03-16 16:55:55 星期六
萧逸小杨*