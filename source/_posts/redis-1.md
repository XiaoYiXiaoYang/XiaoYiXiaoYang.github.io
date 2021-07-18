---
title: Redis基本类型与使用
date: 2020-07-22 22:08:30
tags: [Redis]
categories: [学习笔记]
---

<center>
老规矩，在学习一门新的科目的时候，先来看点视频课入个门...
</center>

<!--more-->



### NoSQL



关系数据库 RDBMS

NoSQL数据库  Not Only SQL   non-relational 非关系型数据库



关系数据库瓶颈：

1. 无法应对每秒上万次的读写请求，
2. 无法通过简单的增加硬件来提高性能



NoSQL优势：

1.大数据量读写性能

2.灵活的数据模型

3.高可用



NoSQL劣势：

1.不支持标准的SQL

2.没有关系型数据库的约束

3.没有事务，

4.没有丰富的数据类型





### Redis安装与使用



Redis是当今非常流行的基于KV结构的作为Cache使用的NoSQL数据库

作用：

1.数据库，对数据执行CURD

2.作为Cache使用，提供数据查询性能

一般：user发起请求--> 控制器 --> Service --> Dao --> 数据库

有Redis且Redis有数据： user发起请求--> 控制器 --> Service --> Redis

有Redis但Redis没数据  ：user发起请求--> 控制器 --> Service --> Redis-->Dao-->数据库



3.作为分布式锁



**Redis：**  Remote Dictionary Server



**Key-Value：**Key是字符类型，Value可以是字符串，可以是Map，列表List，集合Set ，有序集合sorted set



**Windows下**

redis.exe  服务端程序

redis.cli  客户端程序

redis.conf  配置文件



启动Redis默认端口6379 

  

第一个命令，创建字符类型的数据

set key value

get key  获取key的值



**Linux下**



1.获取redis的安装

2.上传安装文件

3.解压缩

4.安装gcc 编译redis源文件



redis-server

redis-cli



**启动Redis**

1.前台启动   

Server启动   ./redis-server redis.conf的文件路径

Client启动    ./redis-cli



/usr/local/redis4.0.3/src/redis-server

/user/local/redis4.0.3/redis,conf



2.后台启动

Server启动   ./redis-server redis.conf的文件路径 &

Client启动    ./redis-cli



**关闭Redis**

1. 在Client中  shutdown   

​        客户端发送命令给服务端，服务端拒绝接收新的命令请求，然后将当前处理的命令处理完，关闭服务端服务



2.kill -9 pid



**Redis客户端**

1.发送命令给服务端

2.显示服务端处理结果



*连接*

redis默认连接127.0.0.1上的6379的redis

./redis -h ip -p port   连接其他redis



Redis远程客户端（RDM）   一款图形界面的客户端软件

Redis远程客户端 （Java版）

Redis编程客户端（Jedis）



远程连接需要修改redis主目录下的redis.conf文件

bind ip 绑定ip这一行注释掉

protected-mode yes 保护模式改为no



redis默认有16个数据库 db0-db15





### Redis基本命令



ping   如果返回PONG，表示服务器运行正常



dbsize  返回redis中数据库中key的个数



redis默认16个数据库，可修改conf文件中的值



select db_name    select 5   切换数据库



flushdb  清空当前数据库



quit  exit  退出redis数据库，服务端服务还在运行



**key命令**



keys pattern   查找所有符合模式pattern的key，如keys *、keys wor?d 



快：单线程、内存存储数据、多路复用IO



exists k1 ..  查找有几个列出的key在数据库中，如：exists k1   输出1   exists  ha hb  输出0



expire key second 设置key的存活时间，设置成功返回1，设置失败返回0



ttl key 以秒为单位，返回key的剩余存活时间

返回值-1 没有设置key的存活时间，key永不过期

返回值-2 key不存在 

*验证码例*:输入发送验证码，redis会set yanzhengma 1234 并 expire yanzhengma 60 【60秒后过期】



type key 返回key的类型



del key  删除key，删除不存在的key，返回0；删除存在的key，返回成功删除的数量

例：del k2 k3 k100 k1000



### Redis数据类型及操作命令

redisdoc.com

**字符串类型 （string）**

set key value   value的字符串值最大512M

1.set   key value 

设置成功返回OK

set key value    例： set k1 hello   、  set  k2 “hello world”   【带空格】、set k3 "{"name":"haha"}"



2.get  key 

得到则返回，得不到返回nil



3.incr   key  

将key对应的value存储的数字值+1，如果不存在，则key对应value值先被初始化为0，再+1。 如果key对应value不是数字值，则执行失败



4.decr  key  反过来 -1



5.append  key value   

追加，在字符串后面追加字符串，成功则返回总的字符串长度。

如果key不存在，则创建这个key value



6.strlen key     

返回value字符串的长度，如果key不存在，返回0



7.getrange key start end    

获取子串start到end，包括start也包括end



8.setrange  key offset value 

 从下标offset开始，以value替换，返回修改后的字符串长度。

 如果设置不存在的key，则创建这个key value



9.mset  key1 value1 key2 value2   

设置多对键值对     返回值OK



10.mget key1 key2  

一次获取多个key，值。不存在的key返回nil



**哈希类型 hash**

hash 是一个string类型的field和value的映射表，非常适合存储对象



1.hset key field value

将hash表中key中的域filed字段的值设置为value，如果key不存在，则新建hash表

返回值：如果filed是新的字段，设置成功返回1；如果是旧的字段，设置成功返回0



2.hget key field

获取值，key不存在或field不存在返回nil



3.hmaset key field1 value1 field2 value2



4.hmget key field1 field2



5.hgetall key

以列表形式返回 key域中的所有字段和值

key1

value1

key2

value2



6.hdel key field1 [..field2]

删除字段对应的值，如果没有则忽略

返回成功删除的数量



7.hkeys key



8.hvals key



9.hexists key value 

存在返回1

不存在返回0



10.当删除一个key域中的所有字段后，这个key也就不存在了，为了存储节省资源



**列表类型 List**

左侧操作：l

右侧操作：r

1.lpush key value [..value2]

可以存储重复数据，按照插入顺序排序；如果key不存在，则创建

返回新列表长度

lpush words a b c；则存储为c b a。因为按序插入，每一个插入到最左边



2.rpush key value [..value]



3.lrange key start end



4.llen key

获取列表key的长度



5.lrem key count value   

count >0 从左到右开始删除

count < 0 从右到左开始删除



6.lset key index value 



7.linsert key BEFORE AFTER pivot value

插入到key中值为pivot的之前或之后

插入成功返回新列表长度，失败返回-1



8.lpop key

删除成员，删除成功返回删除的value



9.rpop





**集合Set**

无序集合，成员唯一，不能重复



1.sadd key value [value..]

返回值，新的添加成功的个数。如果添加的值已经存在，则忽略



2.smembers key

查询并显示key对应的value。如果key不存在视为空集合，返回empty list or set



3.sismember

判断member是否是key集合中的成员

返回是为1，否为0



4.scard key

返回key对应集合中元素个数，如果key不存在，返回0



5.srem key value [value..]

删除key集合中的成员，返回删除成功的个数



6.srandmember key [count]

随机从key集合中抽取count个成员。count大于0，则抽取的元素不重复；count小于0，则抽取的元素可能重复



7.spop key [count]

随机从集合key中删除count个元素

返回被删除的元素





**有序集合类型zset**

经过排序的set集合

每个元素关联一个分数，分数可以重复，元素不能重复，

元素按照分数从小到大排序

分数相同时按照value的字典顺序排列



1.zadd key score value [score value..]

添加一个或多个元素到有序集合中，

返回值：新添加的元素个数



2.zrange key start stop [WITHSCORES]

返回区间start到stop

选项WITHSCORES 为显示score value一同返回  返回到一个列表里

例：

zhangsan

60

lisi

80



3.zrevrange key start  stop [WITHSCORES]

按分数值从大到小来显示



4.zrem key value [value..]



5.zrangebyscore key min max [WITHSCORES] [LIMIT offset count]

获取分数值在min 到 max之间  

LIMIT是分页显示的

可以在min max上带小括号 为不包含的意思



可以借助参数 -inf   表示当前集合中的最小值

可以借助参数+inf 表示当前集合中的最大值



LIMIT offset count   从偏移offset开始  显示count个



6.zrevrangebyscore key min[WITHSCORES] [LIMIT offset count]

从大到小显示



7.zcount key min max

统计区间中成员个数



只要能把自己想排序的东西变成数字的排序，就可以使用有序集合去使用







