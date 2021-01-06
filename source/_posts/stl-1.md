---
title: STL-空间配置器
date: 2019-12-25 14:04:13
tags: [STL]
categories: [学习笔记]
---

 总是隐藏在容器后面默默工作的

<!--more-->

SGI STL在这个项目上逃脱了STL标准规格，使用一个专属的、拥有次层配置能力的、效率优越的特殊配置器。

### SGI的标准空间配置器 std::allocator

SGI未使用过，原因是效率不佳

std:allocator只是简单的对 ::operator new 和 operator delete做了一层简单的包装

### SGI的特殊空间配置器 std::alloc


```
class Foo{...}

Foo *pf = new Foo;  //配置内存，然后构造对象
delete pf;          //将对象析构，然后释放内存
```

其中new运算符包含

1.调用::operator new配置内存

2.调用Foo::Foo构造对象内容


delete也包含

1.调用Foo::~Foo析构对象内容

2.调用::operator delete配置内存


SGI STL将内存配置由alloc::allocate()负责，内存释放由alloc::deallocate()负责

对象构造由::construct()负责 对象析构由::destroy()负责

**插曲**

上述的::operator delete和::operator new是系统提供的全局函数

new和delete是用户进行动态内存申请和释放的操作符

new在底层调用operator new全局函数来申请空间，delete在底层通过operator delete全局函数来释放空间。


**construct() 和 destroy()**

#### construct

```
template <class T1, class T2>
inline void _construct(T1* p, const T2& value) {
    new(p) T1(value);           // placement new. 调用T1::T1(value)
}
```

上述构造又多出一个new

这个new是placement new

是重载operator new 的一个标准、全局的版本，它不能够被自定义的版本代替

```
void *operator new( size_t, void * p ) throw() { return p; }
```

placement new的执行忽略了size_t参数，只返还第二个参数。其结果是允许用户把一个对象放到一个特定的地方，达到调用构造函数的效果。

**例：**

```
pi = new (ptr) int; pi = new (ptr) int;     //placement new
```

括号里的参数ptr是一个指针，它指向一个内存缓冲器，placement new将在这个缓冲器上分配一个对象。Placement new的返回值是这个被构造对象的地址(比如括号中的传递参数)。

**总结**

则 new(p) T1(value) 就是在指针p所指向的内存空间创建一个T1类型的对象，但是对象的内容是从T2类型的对象转换过来的（调用了T1的构造函数，T1::T1(value)）。



#### destroy

多个版本

1.

```
template <class T>
inline void destroy(T* pointer) {
    pointer->~T();   // 直接调用析构函数
}
```

2.

```
template <class ForwardIterator>
inline void destroy(ForwardIterator first, ForwardIterator last) {
  __destroy(first, last, value_type(first));
}
```

版本2调用

```

template <class ForwardIterator, class T>
inline void __destroy(ForwardIterator first, ForwardIterator last, T*) {
  typedef typename __type_traits<T>::has_trivial_destructor trivial_destructor;     // 这里使用了萃取器
  __destroy_aux(first, last, trivial_destructor());
}
```

根据对象有无必要析构调用

```
template <class ForwardIterator>
inline void
__destroy_aux(ForwardIterator first, ForwardIterator last, __false_type) {     // non-trivial destructor
  for ( ; first < last; ++first)
    destroy(&*first);   // 逐个调用析构函数
}
 
template <class ForwardIterator> 
inline void __destroy_aux(ForwardIterator, ForwardIterator, __true_type) {}    // trivial destructor
```

为了效率

首先利用value_type获得迭代器所指对象的类型

再用_type_triaits<T>判断该类型的析构函数是不是无关痛痒

如果是_true_type，则什么也不做就结束

如果是_false_type，才以循环方式访问整个范围，对每个对象调用析构函数



```
// 两个重载版本，无需析构
inline void destroy(char*, char*) {}
inline void destroy(wchar_t*, wchar_t*) {}
```

### 双层级配置器

**第一级**

第一级直接使用malloc() 和 free()

当第一级 malloc 或者 realloc 调用不成功后，

改调用 oom_malloc() 和 oom_realloc() 。

后两者都有内循环，不断调用“内存不足处理例程”，期望在某次调用之后，获得足够的内存。

但如果“内存不足处理例程”未被客户端设定，则直接抛出 bad_alloc 异常，或者终止程序。


**第二级**

当配置区块超过128bytes时，视之为足够大，移交第一级配置器处理，当区块小于128bytes时候，采用内存池管理（memory pool）。

每次配置一大块内存，则维护对应的自由链表（free-list），下次若再有相同大小的内存需求，就直接从 free-list 中拨出（没有就继续配置内存，具体后面讲述），

如果客端释换小额区块，就有配置器回收到 free-list 中


二级配置器使用内存池+自由链表的形式避免了小块内存带来的碎片化，

它是用一个16个元素的自由链表（free_list）来管理的，每个位置的内存大小都是8的倍数，分别为：8、16、24、32、40、48、56、64、72、80、88、96、104、112、120、128。

free_list的节点结构如下：

```
union obj
{
	union obj* free_list_link;
	char client_data[1];
}；
//使用union是为了节省内存，这样每个节点就不需要额外的指针。
```

**二级配置器内存分配**

1、free_list列表中有空余内存。如果申请3个字节的内存，则所需空间大小提升为8的倍数，然后去 free_list 中查找相应的链表，如果 free_list[i] 不为空，则返回第一个元素，然后把头指针往后移。

2、free_list 列表中没有空余，但内存池不为空。首先检验内存池中的大小是不是比申请的内存大，比如申请20*8的内存，如果足够，则分配相应内存，将其中一个分配给用户使用，其它的挂在相应的 free_list 中。如果内存池不够大，只够几个内存分配，则就分配这几个，把相应的数据返回。如果连一个都不够则执行第三中情况。

3、free_list列表中没有空余，内存池也不够。调用malloc重新分配内存，分配时会多分配一倍的内存，把相应的内存挂到free_list下，剩余的放到内存池中。

4、free_list列表中没有空余，内存池也不够，malloc也失败。则调用一级空间配置器，里面会有循环处理，或者抛出异常。

**二级配置器内存回收**

二级空间配置器中申请的内存被释放时，二级空间配置器将回收的内存插入到对应的 free_list 中

**插曲**

simple_alloc

SGI为alloc包装的接口，使配置器的接口能符合STL规格

其内部都是单纯的转调用




2019-10-24 11:55:11 星期四