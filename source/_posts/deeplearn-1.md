---
title: 深度学习环境搭建
date: 2020-01-08 21:22:13
tags: [深度学习]
categories: [技术分享]
---

 涉及深度学习、Python的问题，环境配置就能羁绊好久

<!--more-->

#### 安装anaconda

选择anaconda3版本，

注意中间选择注册环境变量


打开anaconda3

选择environment，新建环境dlwin36

控制台命令行方式，激活环境，activate dlwin36,成功进入则安装成功


### 安装依赖库

pip install dlib==19.6.1 -i https://pypi.douban.com/simple

pip install face_recognition==1.0.0 -i https://pypi.douban.com/simple

pip install opencv-python==3.4.3.18 -i https://pypi.douban.com/simple

安装完继续输入python命令，进入python命令行

使用

import dlib
import face_recognition
import cv2

验证依赖库是否安装成功


### 安装并且破解Pycharm

有教程

### pycharm配置anaconda环境

新建工程后，设置工程选择解释的应用，需要选择anaconda下新建的dlwin36中的python.exe







------------

2019-07-09 19:11:45 星期二