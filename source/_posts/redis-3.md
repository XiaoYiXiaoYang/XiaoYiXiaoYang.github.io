---
title: Redis主从复制&安全
date: 2020-08-05 22:49:42
tags: [Redis]
categories: [学习笔记]
---

<center>
主从复制、哨兵、安全设置、Jedis
</center>
<!--more-->



### 主从复制



持久化功能保证了数据不会丢失，但是由于数据是存储在服务器上的，如果服务器故障了，也会导致数据丢失



为了避免单点故障，我们需要将数据复制多份存在不同的服务器上，一台服务器故障了其他服务器可以提供服务



要求一台服务器的数据更新后，同步到其他服务器；这就需要主从复制



**读写分离**

主redis只负责写

从redis只负责读

通过部署多台redis，并在配置文件中指定这几台Redis 的主从关系。主redis负责写入数据，同时把写入的数据实时同步到机器，这种模式比较主从复制，即master/slave



向slave写数据会导致错误



```
#模拟主从复制

#在一台已经安装redis的机器上，运行多个redis应用模拟多个redis服务器，一个master，两个slave

作为master的redis端口6380
作为slave的redis端口6382 6384

新建redis6380.conf，redis6382.conf，redis6384.conf

#修改配置文件，配置主从关系
主
在redis6380.conf文件中加入
include /usr/local/redis-4.0.13/redis.conf   #包含原来的配置内容
daemonize yes                                #yes后台启动应用
port 6380                                    #自定义端口
pidfile /var/run/redis_6380.pid              #自定义文件，表示当前程序的pid
logfile 6380.log                             #日志文件
dbfilename dump6380.rdb                      #rdb文件名称


从
在redis6382.conf文件中加入
include /usr/local/redis-4.0.13/redis.conf   
daemonize yes                                
port 6382                                   
pidfile /var/run/redis_6382.pid              
logfile 6382.log                            
dbfilename dump6382.rdb  
slaveof 127.0.0.1 6380       #表示自己是从机，跟主机ip、主机端口

在redis6384.conf文件中加入
include /usr/local/redis-4.0.13/redis.conf   
daemonize yes                                
port 6384                                   
pidfile /var/run/redis_6384.pid              
logfile 6384.log                            
dbfilename dump6384.rdb  
slaveof 127.0.0.1 6380       #表示自己是从机，跟主机ip、主机端口
```



这样安排即实现了向主机写数据，也会同步到从机

向从机写数据，会出错



从机只负责读数据



```
#模拟故障处理

   6380                      6382                            6384 
当master服务器挂了，需手动将一台slave服务器提升为master服务器，剩下的slave挂至新的master上，

slaveof no one 				#在一台slave服务器上，将slave服务器提升为master服务器

slaveof 127.0.0.1 6382  	#在另一台slave服务器上，将slave挂至新的master上

6382变为主机
6384变为从机    
6380故障   主从结构恢复
```



```
#恢复6380
6380修好了
slaveof 127.0.0.1 6382    #在6380上执行

这时候恢复为一主两从的结构

主 6382
从6380 6384
```



**哨兵**

上述的故障处理是手动发生的

redis有高可用哨兵来完成自动化故障处理主从切换

redis sentinel：**服务端程序**，它用来监控多个redis服务实例运行情况，redis哨兵是在特殊模式下运行的redis服务器，redis哨兵是在多个进程环境下互相协作工作的



三个任务

1.Sentinal不断地检查主服务器和从服务器是否按照正常预期工作

2.提醒：被监控的redis出现问题的时候，Sentinal会通知管理员或其他应用程序

3.自动故障转移：主redis故障时，sentinal会进行故障转移，将一个从服务器升级为新的主服务器，将其他从服务器挂载到该主服务器上。同时向客户端提供新的主服务器地址



哨兵模式

多个redis哨兵监视主redis服务器，当多个哨兵认为主redis不能工作，则启动故障处理，

哨兵个数至少三个，奇数个；当有一半以上认为redis不能工作

``` 
sentinal.conf

port 26379		 								#端口
dir /tmp 										#哨兵存储文件目录
sentinel monitor mymaster  127.0.0.1 6379 2  	#mymaster是自定义的主redis名称，主redis的ip，主redis的端口，表示裁决数，多少个哨兵认为主redis故障，就切换
```





```
#配置过程
主现在是6382
从事6380 和 6384

拷贝原有哨兵文件到 sentinel26380.conf，sentinel26382.conf，sentinel26384.conf

修改sentinel26380.conf
port 26380
sentinel monitor mymaster  127.0.0.1 6382 2


修改sentinel26382.conf
port 26382
sentinel monitor mymaster  127.0.0.1 6382 2


修改sentinel26384.conf
port 26384
sentinel monitor mymaster  127.0.0.1 6382 2
```



启动方式

```
在三台机器上分别

./redis-sentinal ../sentinal28380.conf

./redis-sentinal ../sentinal28382.conf

./redis-sentinal ../sentinal28384.conf
```



哨兵之前可以交换监控信息

当其中主redis挂掉，哨兵机制会自动切换主redis 

