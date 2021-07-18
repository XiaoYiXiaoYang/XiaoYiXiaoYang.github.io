---
title: tcmalloc 高效的内存分配器
date: 2020-12-04 22:15:48
tags: [TCMalloc]
categories: [学习笔记]
---

<center>
优点、内存分配、内存释放、性能调优
</center>
<!--more-->

tcmalloc 全称 thread cache malloc，即线程缓存的malloc

tcmalloc 是gperftools的一部分，其包括heap-checker，heap-profiler，cpu-profiler



### 优点

快速：tcmalloc做一对malloc/free需要50ns，而glibc做一对malloc/free需要300ns。因为大多数情况的分配和释放不需要锁，多线程程序的争用性低且易扩展

灵活使用内存：可以将释放的内存重新用于分配其他不同大小的对象，或者归还给OS

将相同大小的对象分配到同一page，降低每个对象的内存开销，让小尺寸的对象高效表示

低开销采样：可以深入了解应用程序的内存使用情况



### 内存分配和释放

tcmalloc将内存分配分为3类

小对象：0-256KB

中等对象：256KB-1MB

大对象：1MB-



#### 小对象分配

**size-class**：对于小于等于256kb的对象，tcmalloc将其分为88个类别的class，称为size class，每个class对应大小

16字节以内，每8字节划分为1个size class

16-128字节，每16字节划分为1个size class

128-256字节，按照每次步长size/8字节的长度划分，并且步长需要向下对齐到的整数次幂

![](smallobj1.png)

**Thread Cache**：对于每个线程tcmalloc为其保存了一份单独的缓存，称之为Thread Cache。每个thread cache中为每个size class 都有一个单独的freelist。



小对象的分配：（1）我们将其大小映射到相应的size class。（2）在thread cache中的相应freelist中查找。（3）如果freelist不为空，则从列表中删除第一个对象并返回它。

如果freelist为空：（1）我们从该size class的central cache freelist中获取一堆对象（中央空闲列表由所有线程共享）。（2）将它们放在线程本地freelist中。（3）将新获取的对象之一返回给应用程序。

如果central cache freelist也为空：（1）我们从page heap分配一系列页面。（2）将页面分成一组该大小级别的对象。（3）将新对象放置在central cache freelist上。（4）和以前一样，将其中一些对象移动到线程本地freelist中。



各线程的thread cache连接成一个双向链表。



**Central Cache**：这是一个所有线程公用的缓存，CentralCache中对每个size class也有一个单独的链表来缓存空闲链表，称为centralfreelist，供各线程的thread cache从中取空闲对象。由于是线程公用的，所以各线程访问时需要加锁。



**PageHeap**：当central cache中内存不足，则central cache向page heap申请内存，将其拆分成一系列空闲对象，添加到对应size class的central freelist中。



#### 小对象释放

小对象释放，即确定大小，加入对应的freelist中，如果超过freelist大小，则回收至central cache。

**thread cache freelist大小**

如果freelist太小，我们将需要经常访问central cache freelist。如果freelist太大，则由于freelist中的对象处于空闲状态，我们将浪费内存。

为了适当地调整free list的大小，我们使用慢启动算法来确定每个freelist的最大长度。随着freelist的使用更加频繁，其最大长度也会增加。但是，其freelist最大长度只会增加到max_size。超过max_size则触发内存回收。

```
Start each freelist max_length at 1.

Allocation
  if freelist empty {
    fetch min(max_length, num_objects_to_move) from central list;
    if max_length < num_objects_to_move {  // slow-start
      max_length++;
    } else {
      max_length += num_objects_to_move;
    }
  }

Deallocation
  if length > max_length {
    // Don't try to release num_objects_to_move if we don't have that many.
    release min(max_length, num_objects_to_move) objects to central list
    if max_length < num_objects_to_move {
      // Slow-start up to num_objects_to_move.
      max_length++;
    } else if max_length > num_objects_to_move {
      // If we consistently go over max_length, shrink max_length.
      overages++;
      if overages > kMaxOverages {
        max_length -= num_objects_to_move;
        overages = 0;
      }
    }
  }
```



当thread cache的大小增长到max_size，则thread会向其他不活跃的thread窃取资源。



**thread cache和central cache移动空闲对象**

ThreadCache会从CentralCache中获取空闲对象，也会将超出限制的空闲对象放回CentralCache。ThreadCache和CentralCache之间的对象移动是**批量**进行的，即一次移动多个空闲对象。CentralCache由于是所有线程公用，因此对其进行操作时需要加锁，一次移动多个对象可以**均摊锁操作的开销**。

TCMALLOC_TRANSFER_NUM_OBJ 控制thread cache向central cache移动空闲对象的数量



**thread cache大小**

所有线程的ThreadCache的总大小限制默认是32MB

每个线程的ThreadCache的大小限制默认为4MB

可以通过TCMalloc_MAX_TOTAL_THREAD_CACHE_BYTES环境变量调节或者

```
MallocExtension::instance()->SetNumericProperty("TCMalloc.max_total_thread_cache_bytes", value);
```

调整ThreadCache总大小时，会修改每个ThreadCache的大小限制到512KB~4MB之间的相应值。



#### 中等对象分配

中等对象被四舍五入到页面大小（页面默认8K），由page heap管理，central page heap包含128个freelist数组。这里引入span的概念，**Span**：一个或多个连续的Page组成一个Span。

![](midobj1.png)

从k个page的span链表开始，到128个page的span链表，按顺序找到第一个非空链表。

取出这个非空链表中的一个span，假设有n个page，将这个span拆分成两个span：

- 一个span大小为k个page，作为分配结果返回。
- 另一个span大小为n - k个page，重新插入到n - k个page的span链表中。

如果找不到非空链表，则将其视为大对象



#### 中等对象释放

按照span管理方式释放

在delete一个span时还会以一定的频率触发释放span的内存给系统的操作（`ReleaseAtLeastNPages()`）。释放的频率可以通过环境变量`TCMALLOC_RELEASE_RATE`来修改。**默认值为1，表示每删除1000个page就尝试释放至少1个page**，2表示每删除500个page尝试释放至少1个page，依次类推，10表示每删除100个page尝试释放至少1个page。0表示永远不释放，值越大表示释放的越快，合理的取值区间为[0, 10]。

释放规则：

- 从小到大循环，按顺序释放空闲span，直到释放的page数量满足需求。
- 多次调用会从上一次循环结束的位置继续循环，而不会重新从头（1 page）开始。
- 释放的过程中能合并span就合并span
- 可能释放少于num_pages，没那么多free的span了。
- 可能释放多于num_pages，还差一点就够num_pages了，但刚好碰到一个很大的span。



#### 大对象分配

超过1MB的对象，认为是大对象分配。其策略是直接向page heap申请一块k page大小的span。page heap有个按span大小排序的有序set缓存，其搜索过程如下：

搜索set，找到不小于k个page的最小的span（**best-fit**），假设该span有n个page。

将这个span拆分为两个span：

- 一个span大小为k个page，作为结果返回。
- 另一个span大小为n - k个page，如果n - k > 128，则将其插入到大对象缓存的set中，否则，将其插入到对应的中等对象freelist链表中。

如果找不到合适的span，则使用sbrk或mmap向系统申请新的内存以生成新的span，并重新执行中对象或大对象的分配算法。

总览：

![](allobj.png)



#### 大对象释放

和中等对象释放一个原理，delete span后触发。



### 细节



#### Span的三种状态

IN_USE、ON_NORMAL_FREELIST、ON_RETURNED_FREELIST

inuse 就是tcmalloc正在使用中

on_normal_freelist就是加入到tcmalloc的空闲列表中，等待应用程序再次使用

on_returned_freelist就是已经被tcmalloc归还给操作系统，在linux中，tcmalloc使用madvise函数实现该功能。但是即使归还给系统，其虚拟内存地址依然是可访问的，只是对这些内存的修改丢失了而已，在下一次访问时会导致page fault以用0来重新初始化。



#### 每thread一个cache

这依赖于两种技术：Thread Local Storage（TLS)，和Thread Specific Data（TSD）。两者的功能基本是一样的，都是提供每线程存储。TLS用起来更方便，读取数据更快，但在线程销毁时TLS无法执行清理操作，而TSD可以，因此TCMalloc使用TSD为每个线程提供一个ThreadCache，如果TLS可用，则同时使用TLS保存一份拷贝以加速数据的访问。

TLS和TSD的具体细节可参考[《The Linux Programming Interface》](https://links.jianshu.com/go?to=http%3A%2F%2Fman7.org%2Ftlpi%2F)相关章节（31.3，31.4），本文不再展开讨论。

详细可参考源码中`ThreadCache::CreateCacheIfNecessary()`函数和`threadlocal_data_`变量相关代码。



#### 每cpu tcmalloc

可参考https://google.github.io/tcmalloc/design.html



#### PageMap

pagemap缓存了pageid到span的对应关系





### 调优

调节逻辑页大小。page的默认大小为8KB，可在configure时通过选项调整为4-256。

```
./configure <other flags> --with-tcmalloc-pagesize=32 (or 64)
```



调用release api，原理是将page heap freelist中的内存归还给OS

```
MallocExtension :: instance（）-> ReleaseFreeMemory（）;
```



设置release_rate,取值0，10

```
MallocExtension :: instance（）-> SetMemoryReleaseRate（）;
```



TCMALLOC_SMALL_BUT_SLOW模式，会减少很多一开始的缓存

```
./configure <normal flags> CXXFLAGS=-DTCMALLOC_SMALL_BUT_SLOW
```



设置decommit为true，当开启了`aggressive_decommit_`后，删除normal状态的span时会尝试将其释放给系统，释放成功则状态变为returned。

在合并时，如果被删除的span此时是returned状态，则会将与其相邻的normal状态的span也释放给系统，然后再尝试合并。

```
MallocExtension::instance()->SetNumericProperty("tcmalloc.aggressive_memory_decommit", value)
```



更多调优手段参考：https://google.github.io/tcmalloc/tuning.html



### 参考文章

https://www.jianshu.com/p/11082b443ddf

https://gperftools.github.io/gperftools/tcmalloc.html

https://google.github.io/tcmalloc/design.html

https://google.github.io/tcmalloc/tuning.html