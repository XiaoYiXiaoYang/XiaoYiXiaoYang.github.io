---
title: 读者-写者问题
date: 2019-12-22 19:31:22
tags: [操作系统]
categories: [学习笔记]
---

经典进程同步问题

<!--more-->



## 读者-写者问题

有一个许多进程共享的数据区，这个数据区可以是一个文件或者主存的一块内存空间，甚至可以是一组处理器或者寄存器，有一些只读取这个数据区的读进程和只写数据区的写进程，还满足：

1. 任意多的读进程可以同时读这个数据区
1. 一次只有一个写进程向数据区写数据
1. 如果一个写进程正在向写进程写数据，则禁止任何读进程读文件

### 读者优先

```
写者互斥于写者，也互斥于其他读者，则设置写互斥信号量wmutex
读者----挑选第一个读者和写者互斥，同用写互斥信号量wmutex
	----读者之间设置计数变量readcount，读者之间互斥访问readcount，所以设置一个读互斥信号量rmutex
```

读-读 可并行
读-写 互斥
写-写 互斥



```
读者到来
	若是第一个读者，则若无读者读，则开始读，有写者写则插入wmutex等待
	若不是第一个读者，若有读者读，则直接读，若无读者读，则插入rmutex等待
写者到来
	若是无读者读，无写者写，则开始写
	若有读者读或者由写者写，则插入wmutex等待
```


```c++
semaphore rmutex=1,wmutex=1;
int readcount=0; //读者进程计数变量
void write(){
	while(1){
		wait(wmutex);
		//执行写操作
		signal(wmutex);
	}
}

void read(){
	while(true){
		wait(rmutex);
		if(readcount==0)	wait(wmutex);
		readcount++;
		signal(rmutex);
		
		//执行读操作
		
		wait(rmutex)
		readcount--;
		if(readcount==0)	signal(wmutex);
		signal(rmutex);
	}
}

void main(){
	reader();	reader;...reader();
	write();	write();...write()
}
```

### 消除读者优先

一旦有写者到达，则后续的写者必须等待（无论当时是否有读者在读）

如果读者到来
	有写者写或有写者等，则新读者等待
	无写者写或无写者等，则新读者可读
如果写者到来
	无读者读且无写者写，则新写者可写
	有读者读或有写者写，则新写者等待


### 公平型读者-写者

先来先服务（FCFS）
读者和写者都通过一个信号量竞争申请，然后对通过的读者和写者分别使用各自的信号量管理，本批读者和第一个读者看齐，依然可以同时读，但是读写互斥，写写互斥。 

```c++
semaphore rmutex=1,wmutex=1,S=1;  //增加读者写者竞争信号量
int readcount=0; //读者进程计数变量

void write(){
	while(1){
		wait(S)
		wait(wmutex);
		signal(S)
	
		//执行写操作
		
		signal(wmutex);
	}
}

void read(){
	while(true){
		wait(S);	//申请S
		wait(rmutex);
		if(readcount==0)	wait(wmutex);
		readcount++;
		signal(rmutex);
		signal(S); //释放s信号量
		
		//执行读操作
		
		wait(rmutex)
		readcount--;
		if(readcount==0)	signal(wmutex);
		signal(rmutex);
	}
}

void main(){
	reader();	reader;...reader();
	write();	write();...write()
}

```

### 写者优先 

写者考虑成一个群体，让第一个写者去和读者竞争执行权，
设置写者计数变量writecount，且写者互斥访问的需要信号量mutex
读-读不互斥
读-写互斥
写-写互斥

```c++
semaphore rmutex=1,wmutex=1,S=1,mutex=1;  //增加mutex保护writecount
int readcount=0,writecount=0;	//增加写者计数变量 

void write(){
	while(1){
		wait(mutex)
		if(writecount==0)	wait(S)
		writecout++;
		signal(mutex)
		wait(wmutex);
	
		//执行写操作
		
		signal(wmutex);
		wait(mutex)
		writecount--;
		if(writecount==0)	signal(S)
		signal(mutex)
	}
}

void read(){
	while(true){
		wait(S);	//任何一个读者来了都要和第一个写者竞争
		wait(rmutex);
		if(readcount==0)	wait(wmutex);
		readcount++;
		signal(rmutex);
		signal(S); //释放s信号量
		
		//执行读操作
		
		wait(rmutex)
		readcount--;
		if(readcount==0)	signal(wmutex);
		signal(rmutex);
	}
}

void main(){
	reader();	reader;...reader();
	write();	write();...write()
}
```

### 读者数目限定的 读者写者问题

最多允许RN个读者同时读，引入信号量mx
借助于信号量集Swait(S1,t1,d1)  进程对资源的测试要求Si >= ti   否则不予分配，一旦允许分配，则分配的值是di

引入信号量rMax，赋初值RN
引入wmutex保证写者之间互斥


```
int RN；  //最多允许RN个读者同时读
semphore rMax  = RN，wmutex=1；

void reader(){
	Swait(rMax,1,1);
	Swait(wmutex,1,0);
	
	//读操作
	
	Signal(rMax,1);
}

void writer(){
	Swait(wmutex,1,1,rMax,RN,0);
	
	//写操作
	
	Signal(wmutex,1);
}
```

直接基于记录型信号量实现

资源信号量申请在前
互斥信号量申请在后

```
semaphore rmutex=1,wmutex=1,rmax = RN;  //增加啊rmax资源信号量
int readcount=0; //读者进程计数变量
void write(){
	while(1){
		wait(wmutex);
		//执行写操作
		signal(wmutex);
	}
}

void read(){
	while(true){
		wait(rmax);
		wait(rmutex);
		if(readcount==0)	wait(wmutex);
		readcount++;
		signal(rmutex);
		
		//执行读操作
		
		wait(rmutex)
		readcount--;
		if(readcount==0)	signal(wmutex);
		signal(rmutex);
		signal(rmax);
	}
}

void main(){
	reader();	reader;...reader();
	write();	write();...write()
}
```



------------

*2019-05-09 15:54:25 星期四
萧逸小杨*