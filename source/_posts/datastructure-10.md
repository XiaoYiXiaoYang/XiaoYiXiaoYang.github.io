---
title: 单链表反转
date: 2019-12-25 11:23:09
tags: [算法]
categories: [学习笔记]
---

 单链表简单，但是需要思路支持，代码量上来了才能理解

<!--more-->

有结构体
struct Node{
	int data,
	Node *next,
}*node,

void reverse(node * head){

}


p  q  r

首先进入要思考，传入的是头结点，反转后头结点变身尾结点，应该先把头结点处理一下

```
p = head
q =head->next
head->next = NULL
```


再借助于循环求解

```
while(q->next!=NUll){

 if(NULL==head|| NULL==head->next) return head;    //少于两个节点没有反转的必要。  
    ActList* p;  
    ActList* q;  
    ActList* r;  
    p = head;    
    q = head->next;  
    head->next = NULL; //旧的头指针是新的尾指针，next需要指向NULL  
    while(q){  
        r = q->next; //先保留下一个step要处理的指针  
        q->next = p; //然后p q交替工作进行反向  
        p = q;   
        q = r;   
    }  
    head=p; // 最后q必然指向NULL，所以返回了p作为新的头指针  
    return head;      
}
```