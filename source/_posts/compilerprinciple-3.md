---
title: 编译原理（第三章 语法分析）
date: 2020-01-08 21:02:15
tags: [编译原理]
categories: [学习笔记]
---

 任务是把词法分析得到的词法记号流分析输出分析树的某种中间表示

<!--more-->

#### 第三章 语法分析

**上下文无关文法**

四元组   G（Vt,Vn,S,P）
Vt终结符集合
Vn是非终结符集合
S是一个非终结符，称为开始符号
P是产生式的有限集合

**推导**：将产生式看成是重写规则，把符号串中的非终结符用其产生式右部的串来代替

**分析树**：分析树是推导的图形表示

**二义性**：一些句子存在不止一棵分析树，或者说这些句子存在不止一种最左推导

文法 E -> E + E | E * E | (E) | -E | id
句子：id * id + id

```
推导1：E =》 E * E  =》id * E  =》 id * E + E =》 id * id + id 

推导2：E =》 E + E =》 E * E + E =》 id * E +E => id *id + id
```

**正规式到上下文无关文法**

机械的把一个NFA变换成一个上下文无关文法，为NFA的每个状态i引入非终结符Ai，其中A0是开始符号，
如果状态i有一个a转换到状态j，则Ai---->aAj;
如果是空串 € 转换，则引入Ai----->Aj；
如果i是接收状态，则引入Ai-----> €

**消除二义性**

通过重写文法消除二义性

```
stmt -> if expr then stmt
      | if expr then stmt else stmt
	  | other

重新改写为

stmt -> matched_stmt
      | unmatched_stmt

matched_stmt ->if expr then matched_stmt else  matched_stmt
				| other
unmatched_stmt -> if expr then stmt
				| if expr then matched_stmt else unmatched_stmt
```

**消除左递归**

直接左递归	A->A@     

左递归产生式 A-> A@ | &    (阿尔法  贝塔？？)

A -> &A'
A'->@ |A'|€


非直接左递归
S -> Aa | b
A-> Sd  | €

**提左因子**

当不清楚应该用哪个选择来替换他时，可以通过重写文A产生式来推迟这种决定

A -> @&1 | @&2

A->@A'
A'->&1 | &2


**LL1文法**

FIRST(A)集从左边找到非终结符A,再去看右边

- 产生式以终结符开头，直接加入FIRST集合
- 以非终结符C开头，则把非终结符的FIRST集加入A的FIRST集
- 	如果C的FIRST集有空，则把C后面的D的FIRST集加入A的FIRST集
	

FOLLOW(A)集从产生式找到非终结符A，再去看A的后面

- A是开始文法符号，则将‘$’加入FOLLOW(A)
- A的后面是终结符，则直接加入FOLLOW(A)
- A的后面是非终结符，则把这个非终结符的FIRST集去空，加入FOLLOW(A)
- A处在末尾，则“->”左边符号的FOLLOW集称为A的FOLLOW


满足这俩个条件的**叫LL(1)文法**
对任何两个产生式 A -> @ | &
FIRST(@) 交 FIRST (&) =空
如果& 能零步或多步推导出 € 则 FIRST(@) 交 FOLLOW(A) = 空

**构造LL(1)分析表**

对文法的每个产生式A—>@,执行1和2
1.对FIRST(@)的每个终结符a，把A->@加入M[A,@]
2.如果  € 在FIRST(@)中，对FOLLOW(A)的每个终结符b(包括$)，把A->@加入M[A,b]

**SLR、LR、LALR**

圈圈关系：


**构造SLR分析表**

构造文法G'的LR(0)项目集规范族 C={I0,I1...}
状态i从Ii构造，Action函数在状态i的值这么确定
	(a)如果【A->@.a&】在Ii中，并且goto(Ii,a) =  Ij,那么置action[i,a]为sj，其含义是把a和状态j移入栈，此时a必须是终结符
	(b)如果【A->@.】在Ii中，那么对FOLLOW(A)中的所有a，置action[i,a]为rj，j是产生式A->@的编号，
	(c)如果【S'->S.】在Ii中，那么置action[i,$]为接受acc

goto函数这么确定

对所有的非终结符A，如果goto(Ii,A)=Ij,那么goto[i,A]=j


**构造LR分析表**

加入了搜索符

构造文法G'的LR(0)项目集规范族 C={I0,I1...}
状态i从Ii构造，Action函数在状态i的值这么确定
	(a)如果【A->@.a&,b】在Ii中，并且goto(Ii,a) =  Ij,那么置action[i,a]为sj，其含义是把a和状态j移入栈，此时a必须是终结符
	(b)如果【A->@.,a】在Ii中，那么对FOLLOW(A)中的所有a，置action[i,a]为rj，j是产生式A->@的编号，
	(c)如果【S'->S.,$】在Ii中，那么置action[i,$]为接受acc

goto函数这么确定

对所有的非终结符A，如果goto(Ii,A)=Ij,那么goto[i,A]=j

**构造LALR分析表**

能力在SLR和LR之间，LR状态太多，合并同心的项目集


**多说无益，刷题**

3.1 
	S->(L) |a
	L-> L,S | S 
建立句子的分析树
为两个句子构造最左推导和最右推导
这个文法产生的语言是什么？(产生由a和括号匹配产生的一系列句子)


3.2 
S-> aSbS | bSaS | €

构造两个不同的最左推导
为abab构建最右推导
为abab构建对应的分析树
这个文法产生的语言是什么?(产生a和b个数相等的串)

3.3
S -> S and S | S or S | not S | true | false | (S)

为他写一个等价的非二义文法
【E-> E and T | T
  T-> T or F | F
  F-> not F | p | q | (E)
】[注意最后是(E)]


3.10 构造LL1分析表
D-> TL
T-> int | real
L->id R
R->,id R | €

| 符号 | FIRST集  | FOLLOW集合 |
| ---- | -------- | ---------- |
| D    | int,real | $          |
| T    | int,real | id         |
| L    | id       | $          |
| R    | ,,  €    | $          |

填LL1分析表


3.11 类似

S->aBS | bAS | €
A->bAA | a
B-> aBB | b

| 符号 | FIRST  | FOLLOW |
| ---- | ------ | ------ |
| S    | a,b, € | $      |
| A    | a,b    | a,b,$  |
| B    | a,b    | a,b,$  |


3.12，证明不是LL1文法，找两个|的，求FIRST交集不为空则不是LL1


3.17 慢慢做 构造项目同心集，，，
S->(L) |a
L->L,S |S

3.19 SLR分析表，慢慢做(填表的时候也需要FIRST,FOLLOW)
E->E+T |T
T->TF | F
F->F* |a | b

3.20 证明不是LL1，证明是SLR，
S->SA | A
A->a

不是LL1，常规思路，或者含有左递归，不是LL1

是SLR，构造SLR分析表（分析表不大的情况下）；分析语法产生的句子，是一连串a，对第一个输入a，规约为A，再规约为S;对之后的输入a，规约为A，SA再规约为S，没有冲突是SLR

3.21 是LL1，但不是SLR

S->AaAb | BbBa
A-> €
B-> €

是LL1，对|的求first，求交集不为空则是LL1；第二条基本用不上

不是SLR，产生句子ab，ba，在面临第一个输入符号进行空规约，但是FOLLOW(A)和FOLLOW(B)都是{a,b}，所以不知道规约成A还是B，规约规约冲突


3.22 是LALR(1),但不是SLR(1)

S->Aa |bAc |dc |bda
A->d

不是SLR，对FOLLOW(A)={a，c}，面临c时，不知道移进还是dc规约，移进--规约冲突

是LALR，构造LR分析表就没有上述冲突，只有在面临a时，才进行d到A的规约，所以该文法是LR文法，没有同心的项目集，所以该文法也是LALR文法

3.24 是LR(1),但不是LALR(1)

S->Aa | bAc |Bc |bBa
A->d
B->d

是LR1:该文法只产生da，bdc，dc，bda句子，所d在什么情况下规约成A，d在什么情况下规约成B，可以分清，是LR1文法

在LR1项目集中，对活前缀d有效的项目集是[A->d.,a][B->d.,c]
对活前缀bd有效的项目集是[A->d.,c][B->d.,a]

合并之后[A->d.,a|c][B->d.,a|c]出现规约规约冲突，不是LALR1

好像都有可能手撕

那就再写一遍，当是押题

------------