---
title: wsl (安装与使用)
date: 2021-06-21 22:34:43
tags: [WSL]
categories: [学习笔记]
---

<center>
1.windows开启wsl功能
2.安装LxRunOffline 和 centos8
3.命令使用
</center>
<!--more-->

## windows开启WSL支持

1.windows -- 启用或关闭winwos功能 -- 适用于linux的windows子系统 

## 安装LxRunOffline 和 centos8

**安装LxRunOffline**

下载地址：https://github.com/DDoSolitary/LxRunOffline/releases
下载好之后配置bin目录为环境变量
在shell中输入LxRunOffline version验证是否可用

**准备centos8镜像**

github地址：https://github.com/CentOS/sig-cloud-instance-images/tree/CentOS-8-x86_64/

选择centos8分支，进入 docker 目录，下载文件 centos-8-x86_64.tar.xz


**安装centos8**

以管理员启动powershell。执行命令 `LxRunOffline install -n 自定义系统名称 -d 安装目录路径 -f 安装包路径.tar.xz`

```
LxRunOffline install -n centos -d C:/centos -f D:\centos-8-x86_64.tar.xz

LxRunOffline l 
# 查看是否安装成功
```


## 命令使用

**LxRunOffline**

```
# 列出所有已安装镜像
LxRunOffline list

# 运行
LxRunOffline run -n <自定义的系统名>


# 卸载镜像
LxRunOffline uninstall -n <自定义的系统名>
```


**wsl**

```
# 列出已安装wsl信息
wsl -l -v

# 启动wsl
wsl -d <自定义的系统名>

# 设置wsl版本为2
wsl --set-version <自定义的系统名> 2

# 关闭系统
wsl --shutdown -n <自定义的系统名>
```

**目录**

```
# 在windows下访问子系统目录
# 使用的是 WSL 分发版的默认用户。 因此，任何访问 Linux 文件的 Windows 应用都具有与默认用户相同的权限。
\\wsl$\<自定义的系统名>

# 在linux下访问windows父系统目录
# 如果你的 Windows 用户帐户具有对该文件的读取和执行访问权限，但不具有对该文件的写入访问权限，则该帐户将对用户、组和其他对象显示为 r-x。 如果该文件在 Windows 中设有“只读”属性，则我们不会在 Linux 中授予写入权限。
# 当需要对Windows里面的文件进行写操作时，需要使用管理员权限运行WSL，并且使用chmod 777 {文件}修改文件的权限
cd /mnt/{Windows盘符}
```