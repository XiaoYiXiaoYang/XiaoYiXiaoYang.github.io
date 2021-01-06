---
title: UNIX网络编程--·TCP客户服务器
date: 2020-06-06 17:21:37
tags:[UNIX网络编程]
categories:[学习笔记]
---



编写基本的客户/服务器程序，以及分析各种异常终止的情况

<!--more-->

客户和服务器启动时会发生什么？客户正常终止时发生什么？若服务器进程在客户之前终止，则客户会发生什么？若服务器主机崩溃，则客户会发生什么？



#### TCP回射服务器程序



```
#include "unp.h"

int main(int argc, char **argv)
{
	int listenfd,confd;
	pid_t childpid;
	socklen_t clien;
	struct sockaddr_in cliaddr,servaddr;
	
	listenfd = Socket(AF_INET,SOCK_STREAM,0);
	
	bzero(&servaddr,sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);
	
	Bind(listenfd,(SA*)&servaddr,sizeof(servaddr));
	
	Listen(listenfd,LISTENQ);
	
	for(;;)
	{
		clien = sizeof(cliaddr);
        connfd = Accept(listenfd,(SA*)&cliaddr,&clien);
        if((childpid =Fork() == 0))
        {
        	Close(listenfd);
        	str_echo(connfd);
        	exit(x);
        }
        Close(connfd);
	}
    
}
```



在地址结构中填入通配地址INADDR_ANY，和服务器众所周知端口SERV_PORT





```
#include "unp.h"

void str_echo(int sockfd)
{
	ssize_t n;
	char buf[MAXLINE];

again:
	while(( n == read(sockfd,buf,MAXLINE)) > 0)
	{
		Writen(sockfd,buf,n);
	}
	
	if(n < 0 &&errno == EINTR)
    {
		goto again;
	}
	else if(n < 0)
	{
		err_sys("str_echo: read error");
	}
}
```







#### TCP回射客户程序



```
#include "unp.h"

int main(int argc, char **argv)
{
	int sockfd;
	struct sockaddr_in servaddr;
	if(argc != 2)
	{
		err_quit("usage:tcpli <IPaddress>");
	}
	
	sockfd = Socket(AF_INET,SOCK_STREAM,0);
	
	bzero(&servaddr,sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	
	Inet_pton(AF_INET,argv[1],&servaddr.sin_addr);
	
	Connect(sockfd,(SA*)&servaddr,sizeof(servaddr));
	
	str_cli(stdin,sockfd);
	
	exit(0);
}
```





```
#include "unp.h"

void str_cli(FILE *fp,int sockfd)
{
	char sendline[MAXLINE],recvline[MAXLINE];
	while(Fgets(senline,MAXLINE,fp)!= NULL)
	{
		Writen(sockfd,sendline,strlen(sendline));
		
		if(Readline(sockfd,recvline,MAXLINE) == 0)
		{
			err_quit("str_cli: server terminated prematurely");
		}
		
		Fputs(recvline,stdout);
	}
}
```



#### 正常启动



客户调用socket和connect，后者引起三路握手过程，三路握手完成，客户的connect和服务器的accept均返回。

接着步骤：

1. 客户调用str_cli函数，函数阻塞于fgets调用，等待用户输入
2. 服务器调用fork，子进程调用str_echo，该函数调用readline，readline调用read，read等待客户发送文本阻塞
3. 另一方面，服务器再次调用accept阻塞，等待下一个客户连接



#### 正常终止



1. 当我们在客户键入EOF字符时，fgets返回空指针，str_cli函数返回
2. str_cli返回到客户的main函数，main调用exit终止
3. 进程终止处理部分工作是关闭所有打开的描述符，由内核关闭客户打开的套接字时，客户TCP发送一个FIN给服务器，服务器则以ACK响应，客户向服务器已经关闭，还需要服务器向客户关闭
4. 服务器TCP接收FIN，服务器子进程阻塞于readline调动，readline返回0，导致str_echo函数返回服务器子进程的main函数
5. 服务器子进程通过exit终止
6. 服务器子进程打开的描述符也关闭，引发服务器TCP套接字连接终止序列的最后两个分节【一个服务器到客户的FIN和一个从客户到服务器的ACK】
7. 进程终止的另一部分内容：服务器子进程终止时，给父进程发送一个SIGCHLD信号，本例没有在代码中捕获该信号，该信号的默认行为时被忽略，既然父进程未加处理，子进程于是进入僵死状态。



#### POSIX信号处理



信号就是告知某个进程发生了某个时间的通知，有时也成为软中断



信号可以：

由一个进程发给另一个进程

由内核发给某个进程



我们通过调用sigaction函数来设定一个信号的处置，三种选择

1. 可以提供一个函数，只要有特定信号，他就被调用，这就是信号处理函数，行为叫捕获 【两个信号不能被捕获，是SIGKILL和SIGSTOP】
2. 可以把信号的处置设定为SIG_IGN来忽略，SIGKILL 和 SIGSTOP不能忽略
3. 可以把某个信号的处置设定为SIG_DFL来启用它的默认处置





#### signal函数



第一个参数信号名，第二个参数为指向函数的指针

```
#include "unp.h"

Sigfunc * signal(int signo,Sigfunc *func)
{
	struct sigaction act,oact;
	
	act.sa_handler = func;   /*指向函数的指针*/
	sigemptyset(&act.sa_mask);
	act.sa_flags =0;
	
	if(signo == SIGALRM)
	{
#ifdef SA_INTERRUPT
	act.sa_flags |= SA_INTERRUPT;
#endif
	}
	else
	{
#ifdef SA_RESTART
    act.sa_flags |= SA_RESTART;
#endif
	}
	
	if(sigaction(signo,&act,&oact) < 0)
		return (SIG_ERR);
	return (oact.sa_handler);
}
```







#### POSIX信号语义



- 一旦安装了信号处理函数，他便一直安装着
- 在一个信号处理函数运行期间，正被递交的信号是阻塞的
- 

