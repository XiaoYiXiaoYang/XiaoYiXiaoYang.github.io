---
title: Unix网络编程--TCP套接字编程
date: 2020-03-13 22:06:20
tags: [UNIX网络编程]
categories: [学习笔记]
---

介绍编写一个完整的TCP客户/服务器程序所需要的基本套接字函数

<!--more-->



#### socket函数

为了执行网络I/O，进程必须调用socket函数，指定期望的通信协议类型



```c
int socket(int family,int type,int protocol);
```

family指明协议族，type指明套接字类型，protocol指明协议类型



|  family  |    说明    |
| :------: | :--------: |
| AF_INET  |  IPv4协议  |
| AF_INET6 |  IPv6协议  |
| AF_LOCAL | Unix域协议 |
| AF_ROUTE | 路由套接字 |
|  AF_KEY  | 秘钥套接字 |



|      type      |      说明      |
| :------------: | :------------: |
|  SOCK_STREAM   |  字节流套接字  |
|   SOCK_DGRAM   |  数据报套接字  |
| SOCK_SEQPACKET | 有序分组套接字 |
|    SOCK_RAM    |   原始套接字   |



|   protocol   |     说明     |
| :----------: | :----------: |
| IPPROTO_TCP  | TCP传输协议  |
| IPPROTO_UDP  | UDP传输协议  |
| IPPROTO_SCTP | SCTP传输协议 |



**对比AF_XXX和PF_XXX**



AF_前缀表示地址族，PF_前缀表示协议族



曾经有想法：单个协议族可以支持多个地址族，PF_值用来创建套接字，AF_值用于套接字地址结构，但是支持多个地址族的协议族没有实现过



#### connect函数



```
int socket(int sockfd,const struct sockaddr *servaddr,socklen_t addrlen);
```



客户在调用connect函数前不必非得调用bind函数，因为如果需要的话，内核会确定源IP地址，并选择一个临时端口作为源端口。



connect函数将触发三次握手过程，

1. 若TCP客户没有收到SYN分节的响应，，则返回ETIMEDOUT错误
2. 若对客户的SYN响应的RST，则表明服务器主机在我们的指定的端口与上没有进程在与之等待
3. 若客户发生的SYN在中间的某个路由器上引发一个"destination unreachable"，则认为是一种软错误



#### bind函数



```
int bind(int sockfd,const struct sockaddr *myaddr,socklen_t addrlen);
```



bind函数把一个本地协议地址赋予一个套接字



调用bind可以指定IP地址或端口，可以两者都指定，也可以都不指定



如果指定端口号为0，那么内核就在bind被调用时选择一个临时端口







#### listen函数



```
int listen(int sockfd,int backlog);
```



listen函数把一个未连接的套接字转换成一个被动套接字，表示内核应接受指向该套接字的连接请求



#### accept函数



```
int accept(int sockfd,struct sockaddr *aliaddr,socklen_t *addrlen);
```



addrlen是一个值-结果参数，调用前addrlen所引用的整数值置为cliaddr所指套接字地址结构的长度，返回时，该整数值即为有内核存放在该套接字地址结构内的确切字节数。



##### fork函数



fork函数调用一次，返回两次

在子进程中返回0

在父进程中返回该进程的进程ID



##### exec函数



exec函数把当前进程映像替换成新的程序文件，而且该新程序通常从main函数开始执行，进程ID并不改变。

我们称调用exec的进程为调用进程，称新执行的程序为新程序。



#### 并发服务器



UNIX中编写并发服务器程序最简单的办法就是fork一个子进程来服务每个客户



我们必须知道每个文件或套接字都有一个引用计数，他是当前打开着的引用该文件或套接字的描述符的个数



#### close函数



```
int colse(int sockfd);
```



colse一个TCP套接字的默认行为是把套接字标记成已关闭，然后立即返回到调用进程，该套接字描述符不能再由调用进程使用，



并发服务器中父进程关闭已连接套接字只是导致相应描述符的引用计数值减去一，既然引用计数值仍大于0，这个colse调用并不引发TCP的四分组连接终止序列



如果父进程对每个由accept返回的已连接套接字都不调用close，那么并发服务器中将会发生什么。首先，父进程最终将耗尽可用描述符，因为任何进程在任何时刻可拥有的打开着的描述符通常都是有限制的。



没有一个客户会被终止，当子进程关闭已连接套接字时，他的引用计数值将由2递减为1，且保持为1，因为父进程永不关闭任何已连接套接字。





#### gethostname和getpeername函数



这两个函数或者返回与某个套接字关联的本地协议地址，或者返回与某个套接字关联的外地协议地址



```
int getsockname(int sockfd,struct sockaddr *localaddr,socklen_t *addrlen);
int getpeername(int sockfd,struct sockaddr *peeraddr,socklen_t *addrlen);

```





这两个函数最后一个参数都是值结果参数



- 在一个没有调用bind的TCP客户上，connect成功返回后，gethostname用于返回由内核赋予该连接的本地IP地址和本地端口号
- 在以端口号为0调用bind后，getsockname用于返回由内核赋予本地的端口号
- 在一个以通配IP地址调用bind的TCP服务器上，与某个客户的连接一旦建立，getsockname就可以用于返回由内核赋予该连接的本地IP地址
- 当一个服务器是由通过accept的某个进程通过调用exec执行程序时，它能够获取客户身份的唯一途径便是调用getpeername

















