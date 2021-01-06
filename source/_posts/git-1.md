---
title: 使用Git将项目上传至github（方法1）
date: 2020-01-08 21:24:43
tags: [Git]
categories: [技术分享]
---

 分享如何将自己满意的项目上传到github

<!--more-->

电脑存储毕竟有限，百度网盘上传那么慢，其实github也不失为一个存储数据的好地方，毕竟我的代码那么烂，应该没人看...曾经学习Java的过程中尝试了使用eclipse上传java基础敲的代码，但是失败了...后来再没折腾

了解到Git也是一款好工具，版本管理器，下载安装，没用过...这次尝试一下，好舒服，以后怕是还会爱上这个工具

1.首先登录我的GitHub账户，（账户好早申请了，但是没上传过东西）
点击New repository新建一个仓库，这次准备上传OpenGL代码绘制的简单地形，

填写仓库名称、仓库描述介绍，剩下的一般不用改，直接创建

Public, Private : 仓库权限（公开共享，私有或指定合作者）

Initialize this repository with a README: 添加一个README.md

gitignore: 不需要进行版本管理的仓库类型，对应生成文件.gitignore

license: 证书类型，对应生成文件LICENSE

2.复制一下地址https://github.com/XiaoYiXiaoYang/terrain.git


3.右键项目，右键会出现两个新选项，分别为Git Gui Here,Git Bash Here,这里选择Git Bash Here，

4.把github上面的仓库克隆到本地
git clone https://github.com/XiaoYiXiaoYang/terrain.git

5.这个步骤以后本地项目文件夹下面就会多出个文件夹，该文件夹名即为github上面的项目名，我多出terrain文件夹，接着把本地项目文件夹下的所有文件（除了新多出的那个文件夹不用），其余都复制到terrain文件夹下，


6.接着继续输入命令 cd terrain，进入terrain文件夹

7.git status 查看仓库状态（都显示绿色表示加入到仓库）

8.git add . （注：别忘记后面的.，此操作是把terrain文件夹下面的文件都添加进来，输入后可再git status查看状态，显示绿色则成功添加了）

9.输入git commit  -m  "提交信息"  

10.输入git push -u origin master   （注：此操作目的是把本地仓库push到github上面，此步骤需要输入github帐号和密码）


这些操作也很简单，一般一次就能成功

------------

*2019-03-16 16:10:38 星期六
萧逸小杨*