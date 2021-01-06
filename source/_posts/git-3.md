---
title: git学习
date: 2020-07-16 22:05:02
tags: [Git]
categories: [学习笔记]
---

参考廖雪峰的官方网站

<!--more-->



#### Git基础

**git init**   通过`git init`命令把这个目录变成Git可以管理的仓库



**git add**  加入暂存区



**git commit** 把暂存区的内容提交到本地分支



**git log**  查看日志



**HEAD**表示当前版本 ；**HEAD^**表示上一个版本 ； **HEAD^^**表示上上一个版本 

**HEAD指针**   指向当前分支





用`git log`可以查看提交历史，以便确定要回退到哪个版本

用`git reflog`查看命令历史，以便确定要回到未来的哪个版本

**git reset --hard <版本号>** 可以去过去和未来的任意版本



工作区和暂存区

.git  文件夹是本地仓库

平级其他目录是工作区

add 到了暂存区



**git checkout --file**   修改的暂时还没add 被撤销



删除文件

```
rm a.txt

git rm a.txt

git commit -m "delete a.txt"
```



添加远程仓库   **git remote add origin git@github.com:michaelliao/learngit.git**

第一次推送到远程 **git push -u origin master**   加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来



从远程库克隆  **git clone git@github.com:michaelliao/gitskills.git**



#### 分支管理



**创建与合并分支**

master分支 、HEAD指针、master指向master分支的最新提交

**新建**：新建dev分支，则dev指向master相同的提交，HEAD指针指向dev；

这时dev分支再修改，dev指针向前移动，master指针不动，

**合并**：dev修改完，把dev合并至master，只需要master指针指向dev的当前提交即可

**删除**：删除dev分支，则删除dev指针



**git checkout -b dev**  新建并切换到dev分支



**冲突**

新建feature1分支，修改第一行并且提交

在master分支，修改第一行且提交

在master分支， git merge feature1 



**分支管理策略**