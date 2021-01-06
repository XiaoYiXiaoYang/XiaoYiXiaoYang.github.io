---
title: 数据结构（线索树）
date: 2019-12-25 11:11:29
tags: [数据结构]
categories: [学习笔记]
---

 分享线索树的构造及使用

<!--more-->

前言：考虑一个具有**n个结点**的二叉链表，在**2n个指针域**中有**n-1个指针域用来存放孩子节点的地址**，存在**n+1个空指针域**，可以利用这些空指针域存放指向该节点在某种遍历序列中的前驱和后继结点的指针。

------------


思考：先来思考这几个数的由来，n个结点，每个结点由两个指针域，左孩子和右孩子，所以有2n个指针域
反向思考：其中n个结点，除了根节点以外，其他结点都作为孩子结点，所以n-1个指针域用来存放孩子结点的地址，则只能剩下n+1个空指针域。
（鬼知道为什么我一直想着有n+1 / 2个叶子结点，其实不一定只有叶子结点左右指针域为空，还有其他结点也有可能）



//线索二叉树
//对任意结点，若做指针域为空，则用做指针域存放该节点的前驱线索，若右指针域为空，则用有指针域存放该节点的后继线索，

//为了区分某节点的指针域存放的是指向前驱后继还是左右孩子，每个结点增设两个标志为 ltag和rtag



```c++
enum flag {Child,Thread}; //枚举常量 Child=0,Thread=1
template <class T>
struct ThrNode
{
    T data;
    ThrNode<T> *lchild,*rchild;
    int ltag,rtag;
};

template<class T>
class InThrBiTree
{
public:
    InThrBiTree(); //构造函数
    ~InThrBiTree(); //析构函数
    ThrNode<T> *next(ThrNode<T> *p); //查找P结点的后继
    void InOrder(); //中序遍历线索链表
private:
    ThrNode<T> *root;  //指向线索链表的头指针
    ThrNode<T> *Creat(ThrNode<T> *bt);  //构造函数调用
    void ThrBiTree(ThrNode<T> *bt,ThrNode<T> *pre); //构造函数调用
};
template<class T>
ThrNode <T> * InThrBiTree<T>::Creat(ThrNode<T> *bt)
{
    char ch;
    cin>>ch;
    if(ch=='#') bt=NULL;
    else
    {
        bt = new ThrNode<T>;bt->data=ch;
        bt->ltag=0;bt->rtag=0;
        bt->lchild = Creat(bt->lchild); //递归建立左子树
        bt->rchild = Creat(bt->rchild); //递归建立右子树
    }
    return bt;
}
/*构造线索
*/
template <class T>
void InThrBiTree<T>::ThrBiTree(ThrNode<T> *bt,ThrNode<T> *pre)
{
    if(bt==NULL)    return;
    ThrBiTree(bt->lchild,pre);//对bt的左指针进行处理
    if(bt->lchild==NULL)  //
    {
        bt->ltag=1;
        bt->lchild=pre;
    }
    if(bt->rchild==NULL)
        bt->rtag=1; //对bt的右指针进行处理
    if(pre!=NULL){
         if(pre->rtag==1)
          pre->rchild=bt; //设置bt的后继线索
    }

    pre = bt;
    ThrBiTree(bt->rchild,pre);
}
/*构造函数
*/
template <class T>
InThrBiTree<T>::InThrBiTree()
{
    root = Creat(root);  //建立带线索标志的二叉链表
    ThrNode<T> *pre;
    pre =NULL; //当前访问结点的前驱结点pre初始化为null
    ThrBiTree(root,pre); //遍历二叉链表，建立线索
}

template <class T>
InThrBiTree<T>::~InThrBiTree(){
    //析构函数

}
//查找后继结点
template<class T>
ThrNode<T>* InThrBiTree<T>::next(ThrNode<T> *p)
{
    ThrNode<T> *q;
    if(p->rtag==1)
    {
        q=p->rchild; //右标志为1，可直接得到后继结点
    }
    else
    {
        q=p->rchild; //工作指针q指向结点p的右孩子
        while(q->ltag == 0){
            q=q->lchild;       //查找最左下结点
        }
    }
    return q;
}

//遍历操作
template<class T>
void InThrBiTree<T>::InOrder()
{
    ThrNode<T> *p;
    if(root==NULL)  return;//线索链表为空，则返回
    p=root;
    while(p->ltag==0) //查找中序遍历的第一个结点p
    {
        p=p->lchild;
    }
    cout<<p->data;
    while(p->rchild!= NULL) //当结点p存在后继，依次访问某后继结点
    {
        p=next(p);
        cout<<p->data;
    }
}

int main()
{
    cout<<"请输入线索树的前序遍历序列："<<endl;
    InThrBiTree<char> tree1;

    cout<<"输出其中序遍历序列为："<<endl;
    tree1.InOrder();
    return 0;
}
```


注意其构造函数InThrBiTree函数首先调用Creat函数建立二叉链表，然后调用ThrBiTree函数线索化该二叉链表

而其查找后继结点Next函数首先查看该结点的右标志是否为1，如果右标志为1，则其右孩子指针指向的就是他的后继结点
若右标志为0，则该节点有右孩子，无法直接找到其后继结点，根据中序遍历操作定义，他的后继结点应该是遍历其右子树时访问的第一个结点，也就是**右子树中的最左下结点**。这时需要沿着右孩子的左指针向下查找，当某节点的左表示为1时，他就是要找的后继结点。

*2019-03-24 15:03:20 星期日
萧逸小杨*

