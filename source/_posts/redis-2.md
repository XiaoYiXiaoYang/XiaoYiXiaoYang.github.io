---
title: Redis事务与持久化
date: 2020-08-02 17:01:55
tags: [Redis]
categories: [学习笔记]
---

<center>
Redis高级话题
</center>

<!--more-->

### 事务

mysql中：一组命令的集合，要么全做要么全部做，即执行和回滚



redis中：也是一组命令的集合，但是redis保证这些命令不会被打断，都会成功



1.multi 

标记一个事务的开始，事务内的多条命令会按照先后顺序被放进一个队列中

返回值：总是ok



2.exec

执行所有事务块内的命令

返回值：事务内所有执行语句内容，事务被打断，返回nil



3.discard

取消事务，放弃事务内块内的所有命令

返回值：总是ok



**事务的实现**

***正常情况***

执行步骤：开启事务，向事务队列中加入命令，最后执行事务的提交



```sql
multi   #开启事务
sadd cars biyadi   #命令入队
sadd cars baoma    #命令入队
exec    #事务执行
```



***事务开始之前，入队命令错误（语法错误；严重错误导致服务器不能正常工作），如内存不足，放弃事务***



```
multi
set k1 1
incr k1 k2   #有语法错误，只能有一个参数
exec         #执行错误，因为有语法错误，放弃执行事务
```





***事务执行exec后，命令执行错误，事务提交***



```
multi
set name zhangsan
lpop name     #这句语法没错，执行会有错，lpop是操作列表的
exec          #执行后lpop name这一句出错，其他的被提交
```

 

***放弃事务***



```
multi
set age 25
set age 30
discard  #放弃事务
```



***watch机制***

mysql在多个事务（T1、T2、T3、T4、T5）操作数据的时候，使用锁的机制去保护

redis使用watch机制



watch机制：使用WATCH监视一个或多个key，跟踪key的value的修改情况。如果有key的value在事务exec执行之前被修改了，整个事务取消。EXEC返回提示信息，表示事务已经失败。

为了高效性，大多数情况下，大家不会同时操作一个键



WATCH机制使得事务EXEC变的所有条件，事务只有在被WATCH的key没有修改的前提下才能执行，不满足条件，事务被取消；使用WATCH监视了一个过期的键，那么即使那个键过期了，事务仍然可以正常执行[???]



(1)WATCH命令可以被调用多次，对键的监视从调用WATCH执行之后开始生效，直到调用EXEC为止，不管事务是都是否成功执行，对所有键的监视都会被取消

(2)当客户端断开连接，该客户端对键的监视也会被取消

(3)UNWATCH可以手动取消对所有键的监视



4.watch

watch key [key..]

监视一个或多个key，如果在事务执行之前这个key被其他命令改动，那么事务被打断

返回值：总是OK



5.unwtatch

取消WATCH命令对所有key的监视，如果在执行WATCH命令之后，EXEC命令或者DISCARD命令被先执行了的话，那么就不需要再执行UNWATCH了





例：

```
A客户端
set num 10
watch num
multi
set num 100
#然后暂停，不执行exec ，开启B客户端

B客户端
set num 200

#然后回到A客户端
A客户端
exec
#返回nil   被监视的num被别的事务修改了，所以本事务放弃提交
```



### 持久化



持久化：可理解为存储，就是将数据存储到一个不会丢失的地方，放在内存中断电重启会丢失。放在磁盘中，就算是一种持久化



Redis数据存储在内存中，如果linux重启或者redis崩溃，都会丢失redis数据，为了解决这种问题，Redis提供两种机制对数据进行持久化存储。



#### 持久化方式



**RDB机制**：在指定时间间隔内将redis内存数据集快照写入磁盘，数据恢复时将快照文件直接再读到内存。



RDB保存了某个时间点的数据集，存储在一个二进制文件中，只有一个文件，默认是dump.rdb，

保存数据是在单独的进程中写文件，不影响Redis的正常使用，RDB恢复数据时比其他AOF速度快



RDB方式的持久化方式，默认启用，在redis.conf文件中配置的。在redis.conf文件中搜索SNAPSHOTTING，进行配置



```
1.配置执行RDB生成快照文件的时间策略
    #在N秒内数据集至少有M个key改动，满足这一策略，自动保存一次数据

配置格式：save <seconds> <changes>
        save 900 1    #900秒内有1个key被改动就做一次快照
        save 300 10
        save 60 10000
       
2.dbfilename   默认是dump.rdb

3.dir 指定RDB文件的存储位置，默认是当前目录
```



缺点：有可能丢失数据，不能100%保证数据全





**AOF机制**（append only file）： Redis每收到一条改变数据的命令时，他就把该命令写到一个AOF文件中，当redis重启时，它通过执行AOF文件中的所有命令来恢复数据



也是需要在refis.conf文件中配置

```
appendonly       #默认是no，改成yes，即开启了AOF持久化
appendfilename   #指定AOF文件名，默认appendonly.aof
dir              #指定RDB和AOF文件存放的目录，默认./
appendfsync      #配置向aof文件写命令数据的策略
  no             #不主动进行同步操作，而是完全交给操作系统来做，每30秒一次，比较快
  always         #每次执行写入都会执行同步，比较慢，但安全
  everysec       #每秒执行一次同步操作，介于速度和安全之间，默认项
  auto-aof-rewrite-min-size  #允许重写的最小AOF文件大小，默认是64M，当aof文件大于64M时，开始整理aof文件，去掉无用的操作命令，缩小aof文件

#resid先把用户的set 等写到内存缓冲区，操作系统每隔30秒写入磁盘
```



```
#模拟aof文件内容
*2
$6
SELECT
$1
0
*3
$3      #$后的数字表示几个字符
set
$2
k1
$2
v1
*3
$3
set
$2
k2
$2
v2
```





总结：RDB性能好，缺点是数据可能丢；AOF性能慢一点，但是可靠

redis先找AOF，如果没有就找RDB恢复