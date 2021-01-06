---
title: Unix网络编程--简介TCP/IP、TCP、UDP、SCTP
date: 2020-03-02 21:30:58
tags: [UNIX网络编程]
categories: [学习笔记]
---



要编写通过计算机网络通信的程序，先要确定通信的程序所用的协议

<!--more-->



- TCP/IP协议族，也称为网际协议族

- 客户和服务器之间的信息流在其中一端是向下通过协议栈的，跨越网络后，在另外一端是向上通过协议栈的



#### 简单的时间获取客户程序



```c
#include "unp.h"

int main(int  argc, char ** argv)
{
    int sockfd,n;
    char recvline[MAXLINE+1];
    struct sockaddr_in servaddr;
    
    if(argc != 2)
    {
        err_quit("usage : a.out <IPaddress>  ");
    }
    
    if((sockfd = socket(AF_INET,SOCK_STREAM,0)) < 0)
    {
        err_sys("socket error");
    }
    
    bzero(&sockfd,sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(13);
    
    if(inet_pton(AF_INET,argv[1],&servaddr.sin_addr) < = 0)
    {
        //argv[1]即是IP地址 inet_pton将其转换为合适的格式
        err_quit("inet_pton error for %S",argv[1]);
    }
    
    if(connect(sockfd,(SA *)&servaddr,sizeof(servaddr)) < 0 )
    {
        err_sys("connect error");
    }
    
    while((n = read(sockfd,recvline,MAXLINE)) < 0)
    {
        recvline[n] = 0;
        if(fputs(recvline,stdout) == EOF)
        {
            err_sys("fputs error");
        }
    }
    
    if(n < 0)
    {
        err_sys("read error");
    }
    
    exit(0);
}

```



#### 错误处理：包裹函数



包裹函数完成实际的函数调用，检查返回值，并且在发生错误的时候终止进程



约定：**包裹函数名是实际函数名的首字母大写形式**



#### Unix errno值



只要一个Unix函数中有错误发生，全局变量errno就被置为指明该错误类型的正值



#### 简单的时间获取服务器程序



```c
#include "unp.h"
#include <time.h>

int main(int  argc, char ** argv)
{
    int listenfd,connfd;
    char buff[MAXLINE+1];
    struct sockaddr_in servaddr;
    time_t ticks;
    
    
    listenfd = Socket(AF_INET,SOCK_STREAM,0)
        
    bzero(&sockfd,sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htol(INADDR_ANY);
    servaddr.sin_port = htons(13); /* daytime server*/
    
    Bind(listenfd,(SA *)&servaddr,sizeof(servaddr));
    
    Listen(listenfd,LISTENQ);
    
    for(;;)
    {
        connfd = Accept(listenfd,(SA*)NULL,NULL);
        
        ticks = time(NULL);
        snprintf(buff,sizeof(buff),"%.24s\r\n",ctime(&ticks));
        Write(connfd,buff,strlen(buff));
        
        Close(connfd);
    }
 
}

```



#### 网络拓扑的发现



- netstat -i 提供网络接口的信息
- netstat -r 显示路由表
- ifconfig eth0 显示每个接口的详细信息





#### 用户数据报协议（UDP）



- 不保证各个数据报的先后数据跨网络后保持不变，也不保证每个数据报只到达一次
- 每个UDP数据报都有一个长度，如果一个数据报正确的到达目的地，那么该数据报的长度将随数据一道传递给接收端应用进程



#### 传输控制协议（TCP）



- TCP是面向连接的
- TCP提供可靠性（确认、重传、序号）
- TCP提供流量控制（TCP总是告知对方在任何时刻它一次能够从端接收多少字节的数据）
- TCP是全双工的（TCP为每个数据流方向跟踪诸如序列号和通告窗口的大小等状态信息



#### 流控制传输协议（SCTP）



- SCTP在客户和服务器之间提供关联，一个关联指代两个系统之间的一次通信，它可能因为SCTP支持多宿而涉及不止两个地址
- SCTP是面向消息的，它提供各个记录的按序递送服务
- SCTP能在所连接的端点之间提供多个流，每个流各自可靠的按序传送消息，互相不干扰
- SCTP支持多宿，单个SCTP端点支持多个IP地址



#### TCP三次握手



- 客户 SYN置1，seq为x
- 服务器 ACK置1，ack为x+1，SYN置1，seq为y
- 客户 ACK置1，ack为y+1,seq为x+1



#### TCP选项



- MSS选项。发送SYN的一端使用本选项告知对端它的最大分节大小
- 窗口规模选。TCP连接任何一端能够告知对端最大窗口大小时65535
- 时间戳选项。



#### TCP连接终止



- 客户A FIN=1,seq=u
- 服务器B ACK=1 ack=u+1，seq=v，  /* 半关闭 */
- 服务器B ACK=1 ack=u+1 seq=w，FIN=1，
- 客户A  ACK=1,seq=u+1，ack=w+1，B收到则关闭连接，A进入TIME-WAIT状态



#### TIME_WAIT状态



- 可靠的实现TCP全双工连接的终止。假设最终客户的ACK丢失，则需要重传，因此客户必须维护状态信息
- 允许来的重复分节在网络中消逝。同样的IP地址和端口上的新连接对于老连接是新的，不会混



#### SCTP四路握手



- 客户SCTP 发送INIT消息   ----->INIT(Ta，J)
- 服务器以一个INIT ACK消息回复         ------>Ta:INIT ACK(Tz,K,cookie C)
- 客户以一个COOKIE ECHO消息回射服务器的状态cookie   ------>Tz:COOKIE ECHO C
- 服务器以一个COOKIE ACK消息确认客户回射的cookie是正确的   ------>Ta:COOKIE ACK



   验证标记 Ta、Tz

​	SCTP四路握手避免拒绝服务攻击



#### SCTP关联终止

当一端关闭某个关联时，另一端也必须停止发送新的数据

- 客户 SHUTDOWN
- 服务器 SHUTDOWN ACK
- 客户 SHUTDOWN COMPLETE



#### SCTP选项



- 动态地址扩展，允许协作的SCTP端点从已有的某个关联中动态增删IP地址
- 不完全可靠性扩展，允许协作的SCTP端点在应用进程的指导下限制数据的重传



#### TCP端口号与并发服务器



主服务器循环通过派生一个子进程来处理每个新的连接

当服务器接收并接受这个客户的连接时，它fork一个自身的副本，让子进程来处理该客户的请求

**已连接套接字使用与监听台阶自相同的本地端口**



- 客户A用端口1500和服务器B连接，监听端口21
- 客户A用端口1501和服务器B连接，



​	则服务器端有三个套接字，一个监听套接字，两个已连接套接字，且已连接的两个套接字有区别，客户的第二个连接使用了新的端口。

​	如果一个分节来自客户A的IP地址，且端口为1500，目的地址为服务器B的IP地址，端口为21，则他被递送给第一个子进程

​	如果一个分节来自客户A的IP地址，且端口为1501，目的地址为服务器B的IP地址，端口为21，则他被递送给第二个子进程



#### 缓冲区大小及限制



- IPv4数据报的最大大小时65535字节
- IPv6数据报的最大大小是65575字节
- 许多网络有一个硬件规定的MTU【最大传输单元】
- 当一个IP数据报将从某个接口送出时，如果它的大小超过相应链路的MTU，IPv4和IPv6都执行分片，IPv4主机对产生的数据报执行分片，IPv4路由器对其转发的数据报执行分片



#### TCP输出



​	每一个TCP套接字有一个发送缓冲区，当某个应用进程调用write时，内核从应用进程的缓冲区复制所有的数据到所写套接字的缓冲区，如果缓冲区容不下所有数据或者缓冲区有数据，则应用进程都将被投入睡眠，直到应用进程缓冲区的数据都复制到套接字缓冲区。

​	从一个write系统调用返回成功仅表示我们可以重新使用原来的应用进程缓冲区，并不表明对端的TCP或应用进程已收到数据。

​	这一端的TCP提取套接字发送缓冲区的数据并且把它发送给对端TCP。伴随不断收到ACK，本端TCP才能从套接字发送缓冲区丢弃已确认的数据



#### UDP输出

​	

​	UDP不必等待确认，没有发送缓冲区

​	UDP套接字的write调用成功返回表示所写的数据或其所有片段已被加入数据链路层的输出队列



#### SCTP输出



​	SCTP是类似TCP的可靠协议，他的套接字也有一个缓冲区

​	内核从应用进程的缓冲区复制数据到SCTP套接字的发送缓冲区



