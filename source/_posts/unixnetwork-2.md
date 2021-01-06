---
title: Unix网络编程--套接字编程简介
date: 2020-03-10 22:04:56
tags: [UNIX网络编程]
categories: [学习笔记]
---



套接字地址结构、地址转换函数。这些结构在内核和进程之间传递

<!--more-->



#### 套接字地址结构



**IPv4套接字地址结构**

```C
struct in_addr
{
    in_addr_t s_addr;
};

struct sockaddr_in
{
    uint8_t sin_len;			/*结构的长度*/
    sa_family_t sin_family;		/**/
    in_port_t sin_port;			/*端口号*/
    struct in_addr sin_addr;	/*IP地址*/
    char sin_zero[8];			/**/
}
```



**通用套接字地址结构**

套接字地址结构总是以引用形式来传递，然而以这样的指针作为参数之一的任何套接字函数必须处理来自所支持的任何协议族的套接字地址结构

```c
struct sockaddr
{
  	uint8_t sa_len;
    sa_family_t sa_family;
    char sa_data[14];
};
```

通用套接字地址结构的唯一用途就是对指向特性与协议的套接字地址结构的指针执行类型强制转换



**IPv6套接字地址结构**



```c
struct in6_addr
{
  uint8_t s6_addr[16];  
};

#define SIN6_LEN

struct sockaddr_in6
{
    uint8_t sin6_len;
    sa_family_t sin6_family;
    in_port_t sin6_port;
    uint32_t sin6_flowinfo;
    struct in6_addr sin6_addr;
    uint32_t sin6_scope_id;
}
```



#### 值-结果参数



​	当往一个套接字函数传递一个套接字地址结构的时候，总是以引用形式传递，**该结构的长度也作为一个参数来传递**。其传递方式取决于从进程到内核，或者从内核到进程



- 从进程到内核的函数有bind、connect、sendto、**传递结构的大小，内核就知道从进程复制多少数据**



```c
connect(sockfd,(SA *)&serv,sizeof(serv));
```





- 从内核到进程的函数有accept、recvfrom、getsockname、getpeernamed。传递指向表示该结构大小的整数变量的指针。当函数被调用时，结构大小时一个值，它告诉内核该结构的大小，这样内核在写该结构的时候不至于越界。当函数返回时，结构大小又是一个结构，它告诉进程该结构的实际大小，即内核写了的



```c
len = sizeof(cli);
getpeername(unixfd,(SA *)&cli,&len);
```



#### 字节排序函数



小端：低位进低地

大端：



**h代表host，n代表network，s代表short，l代表long**



- uint16_t htons(uint16_t host16bitvalue);
- uint32_t htonl(int16_t host32bitvalue);
- uint16_t ntohs(uint16_t net16bitvalue);
- uint32_t ntohl(uint16_t net32bitvalue);



#### 字节操纵函数



​	

```c
void bzero(void *dest, size_t nbytes);
void bcopy(const char *src,void * dest,size_t nbytes);
void bcmp(const void *ptr1,const void *ptr2,size_t nbytes);

void *memset(void *dest,size_t len);
void *memcopy(void *dest,const void *src,size_t nbytes);
void memcmp(const void *ptr1,const void *ptr2,size_t nbytes);
```



#### inet_aton、inet_addr、inet_ntoa函数



```c
int inet_aton(const char* strptr,struct in_addr* addrptr); /*将字符串转换成一个32位的网络字节序二进制值*/

in_addr_t inet_addr(const char *strptr); /*转换为网络字节序二进制，出错时返回INADDR_NONE常值，即255.255.255.255*/

char *inet_ntoa(struct in_addr inaddr); /*将网络字节序二进制值转换为相应的点分十进制数串*/
```



#### inet_pton和inet_ntop函数



**p代表permission，n代表numeric**



```c
int inet_pton(int family,const char *strptr,void *addrptr);/*转换strptr指向的字符串，二进制结果存放在addrptr中*/

const char * inet_ntop(int family,const void* strptr,char * strptr,size_t len);/*len参数表示目标存储单元的带下，防止溢出。 strptr存放转换结果，调用时strptr不能是空指针，调用者必须为其申请足够的内存*/
```



#### sock_ntop和相关函数



inet_ntop函数的一个基本问题是：它要求调用者传递一个指向二进制地址的指针，要求调用者必须知道这个结构的格式和地址族



```c
struct sockaddr_in addr
inet_ntop(AF_INET,&addr.sin_addr,str,sizeof(str));
```



自己编写sock_ntop函数，它以指向某个套接字地址结构的指针为参数，查看该结构内部，然后调用适当的函数返回改地址的表达格式



```c
char * sock_ntop(const struct sockaddr_in *sa,socklen_t salen)
{
    char portstr[8];
    static char str[128];
    
    switch(sa->sa_family)
    {
        case AF_INET:
            struct sockaddr_in *sin = (struct sockaddr_in *)sa;
            
            if(inet_ntop(AF_INET,&sa->sin_addr,str,sizeof(str)) == NULL)
            {
                return NULL;
            }
            
            if(ntohs(sin->sinport) != 0)
            {
                snprintf(portstr,sizeof(portstr),":%d",ntohs(sin->sin_addr));
                strcat(str,portstr);
            }
            return str;
    }
}
```





#### readn、writen和readline函数



​	字节流套接字上调用read或write输入输出的字节数可能比请求的数量少，然而这不是出错的状态，原因在于内核中用于套接字的缓冲区可能已经到达了极限，此时只需要再次调用write或者read函数，以输出剩余的字节。



