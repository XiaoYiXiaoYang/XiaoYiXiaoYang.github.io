---
title: 内存管理器-TCmalloc-官网
date: 2020-10-25 16:00:14
tags: [TCMalloc]
categories: [学习笔记]
---

<center>
https://github.com/google/tcmalloc  翻译：TCMalloc -- 高效的内存分配器
</center>

<!--more-->



## 动机

TCMalloc比我测试过的glibc 2.3 malloc（可称为ptmalloc2的独立库）和其他malloc更快。ptmalloc2在2.8 GHz P4上（用于小型对象）执行malloc / free对大约需要300纳秒。对于同一操作对，TCMalloc实现大约需要50纳秒。速度对于malloc实现非常重要，因为如果malloc不够快，应用程序编写者倾向于在malloc之上编写自己的自定义空闲列表。除非应用程序编写者非常谨慎地适当调整可用列表的大小并从空闲列表中清除空闲对象，否则这可能导致额外的复杂性和更多的内存使用。

TCMalloc还减少了多线程程序的锁争用。对于小对象，竞争几乎为零。对于大型对象，TCMalloc尝试使用细粒度且高效的自旋锁。ptmalloc2还通过使用每个线程的竞技场来减少锁争用，但是ptmalloc2使用每个线程的竞技场存在一个大问题。在ptmalloc2中，内存永远无法从一个领域移动到另一个领域。这会导致大量的空间浪费。例如，在一个Google应用程序中，第一阶段将为其URL规范化数据结构分配大约300MB的内存。当第一阶段完成时，第二阶段将在相同的地址空间中开始。如果为第二阶段分配的舞台不同于第一阶段使用的舞台，此阶段将不重用第一阶段后剩余的任何内存，并将在地址空间中再增加300MB。在其他应用程序中也发现了类似的内存膨胀问题。

TCMalloc的另一个好处是小对象的空间高效表示。例如，可以在使用大约`8N * 1.01`字节的空间时分配N个8字节的对象。即，空间开销为百分之一。ptmalloc2为每个对象使用一个四字节的标头，并且（我认为）将大小四舍五入为8个字节的倍数，最后使用`16N`字节结束





TCMalloc比我测试过的glibc 2.3 malloc（可称为ptmalloc2的独立库）和其他malloc更快。ptmalloc2在2.8 GHz P4上（用于小型对象）执行malloc / free对大约需要300纳秒。对于同一操作对，TCMalloc实现大约需要50纳秒。速度对于malloc实现很重要，因为如果malloc不够快，应用程序编写者倾向于在malloc之上编写自己的自定义空闲列表。除非应用程序编写者非常谨慎地适当调整可用列表的大小并从空闲列表中清除空闲对象，否则这可能导致额外的复杂性和更多的内存使用。

TCMalloc还减少了多线程程序的锁争用。对于小对象，竞争几乎为零。对于大对象，TCMalloc尝试使用细粒度且高效的自旋锁。ptmalloc2也通过使用每个线程的arenas 来减少锁争用，但是ptmalloc2使用每个线程的arenas 存在一个大问题。在ptmalloc2中，内存永远无法从一个领域移动到另一个领域。这会导致大量的空间浪费。例如，在一个Google应用程序中，第一阶段将为其URL规范化数据结构分配大约300MB的内存。当第一阶段完成时，第二阶段将在相同的地址空间中开始。如果为第二阶段分配的arena不同于第一阶段使用的arena，此阶段将不复用第一阶段之后剩余的任何内存，并将在地址空间中再增加300MB。在其他应用程序中也发现了类似的内存膨胀问题。

TCMalloc的另一个好处是小对象的空间高效表示。例如，可以在使用大约`8N * 1.01`字节的空间时分配8N字节的对象。即，空间开销为百分之一。ptmalloc2为每个对象使用一个四字节的标头，并且（我认为）将大小四舍五入为8个字节的倍数，最后使用`16N`字节结束。

总结：快、减少线程锁、小对象高效

### 用法

要使用TCMalloc，只需通过“ -ltcmalloc”链接器标志将TCMalloc链接到您的应用程序中。

您可以通过使用LD_PRELOAD在未编译的应用程序中使用TCMalloc：

```
   $ LD_PRELOAD =“ / usr / lib / libtcmalloc.so” 
```

LD_PRELOAD很棘手，我们不一定推荐这种使用方式。

TCMalloc还包括[堆检查器](https://gperftools.github.io/gperftools/heap_checker.html) 和[堆分析器](https://gperftools.github.io/gperftools/heapprofile.html)。

如果您希望链接一个不包含堆分析器和检查器的TCMalloc版本（也许可以减小静态二进制文件的二进制大小），则可以链接`libtcmalloc_minimal` 来替代 。



### 总览

TCMalloc为每个线程分配一个线程本地缓存。线程本地缓存满足小分配。根据需要将对象从中央数据结构移动到线程本地缓存中，并使用定期垃圾回收将内存从线程本地缓存迁移回中心数据结构中。

![img](https://gperftools.github.io/gperftools/overview.gif)

TCMalloc将大小小于等于256K的对象（“小”对象）与大对象区别对待。使用页面级分配器（页面是内存的8K对齐区域）直接从中央堆分配大对象。即，大对象始终是页面对齐的，并且占据整数页。

可以将一系列页面分成一系列小对象，每个对象大小均相等。例如，一页（4K）可以分成32个对象，每个对象大小为128字节。



### 小对象内存分配

每个小对象尺寸映射到大约88个可分配尺寸等级之一。例如，所有在961到1024字节范围内的分配都被四舍五入为1024。大小类被隔开，以便小尺寸被8字节分隔，大尺寸被16字节分隔，更大尺寸被32字节分隔，依此类推。 。控制最大间距，以便在分配请求刚好超出大小类的末尾并且必须向上舍入到下一个类时，不会浪费太多空间。

线程缓存包含每个大小级别的空闲对象的单链接列表。

![img](https://gperftools.github.io/gperftools/threadheap.gif)

分配小对象时：（1）我们根据其大小映射到相应的类。（2）在线程本地缓存中的相应空闲列表中查找当前线程。（3）如果空闲列表不为空，则从列表中删除第一个对象并返回它。当遵循此快速路径时，TCMalloc根本不会获得任何锁。因为在2.8 GHz Xeon上，锁定/解锁对大约花费100纳秒，所以这大大有助于加速分配。

如果空闲列表为空：（1）我们从该类的中央空闲列表中获取一堆对象（中央空闲列表由所有线程共享）。（2）将它们放在线程本地缓存空闲列表中。（3）将新获取的对象之一返回给应用程序。

如果中央空闲列表也为空：（1）我们从中央页面分配器分配一系列页面。（2）将其分成一组该大小级别的对象。（3）将新对象放置在中央空闲列表上。（4）和以前一样，将其中一些对象移动到线程本地缓存空闲列表中。



### 调整线程缓存空闲列表的大小

正确调整线程缓存空闲列表的大小很重要。如果空闲列表太小，我们将需要经常访问中央空闲列表。如果空闲列表太大，则由于空闲列表中的对象处于空闲状态，我们将浪费内存。

请注意，线程缓存空闲列表对于释放和分配都同样重要。如果没有缓存，则每个重新分配都需要将内存移至中央空闲列表。另外，某些线程具有非对称的分配/释放行为（例如，生产者线程和消费者线程），因此正确地确定空闲列表的大小会变得更加棘手。

为了适当地调整空闲列表的大小，我们使用慢启动算法来确定每个空闲列表的最大长度。随着空闲列表的使用更加频繁，其最大长度也会增加。但是，如果空闲列表用于释放的次数多于申请内存的次数，则其最大长度只会增加到可以立即将整个列表有效地移动到中央空闲列表的程度。

下面的伪代码说明了这种慢启动算法。请注意，`num_objects_to_move`等于每个类的空闲列表长度。通过移动一个对象列表长度，中央缓存可以在线程缓存之间有效地传递这些列表。如果线程缓存希望少于`num_objects_to_move`，则对中央空闲列表的操作具有线性时间复杂度。始终使用`num_objects_to_move`与中央缓存之间来回传输的对象数量的缺点是，它浪费了不需要所有这些对象的线程中的内存。



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



另请参阅“[垃圾回收](https://gperftools.github.io/gperftools/tcmalloc.html#Garbage_Collection)”部分 以了解它如何影响`max_length`。	



### 中等对象内存分配

中型对象大小（256K≤大小≤1MB）被四舍五入为页面大小（8K），并由中心页面堆处理。中心页面堆包含128个空闲列表的数组。第`k`项是页面大小为k+1pages的运行的空闲列表：

![img](https://gperftools.github.io/gperftools/pageheap.gif)

大小为k pages的内存申请通过在第`k`个空闲列表中查找，可以满足页面分配要求。如果该空闲列表为空，则查找下一个空闲列表，依此类推。如果没有任何中型对象的空闲列表不能满足分配，则将该分配视为大对象。



### 大对象内存分配

1MB或更多的分配被认为是大分配。可以满足这些分配要求的可用内存跨度在按大小排序的红黑树中进行跟踪。分配遵循*best-fit* 算法：搜索树以找到最小的可用空间跨度，该跨度大于请求的分配。分配是从该范围中划出的，剩余的空间将重新插入到大型对象树中，或者可能是重新插入到较小的空闲列表之一中。如果没有空闲存储能适合所请求的分配，我们从系统申请内存（使用存储器`sbrk`， `mmap`或者通过在部分的映射 `/dev/mem`）。

如果申请k pages大小的内存>的页面运行满足页面分配，则将运行的其余部分重新插入到页面堆中的相应空闲列表中。



## Spans

TCMalloc管理的堆由一组页面组成。连续页面的运行由一个`Span`对象表示。span可以*分配*，也可以*空闲*。如果空闲，则span是页面堆链接列表中的条目之一。如果已分配，则它要么是已移交给应用程序的大对象，要么是已分成多个小对象序列的一系列页面。如果拆分为小对象，则将对象的大小级别记录在span中。

由页码索引的中央数组可用于查找页面所属的span。例如，下面的span*a*占2页，span*b*占1页，span*c*占5页，span*d*占3页。

![img](https://gperftools.github.io/gperftools/spanmap.gif)

在32位地址空间中，中央数组由2级基数树表示，其中根包含32个条目，每个叶包含2 ^ 14个条目（32位地址空间具有2 ^ 19个8K页，树的第一层将2 ^ 19页除以2 ^ 5）。这导致中央阵列的起始内存使用空间为64KB（2 ^ 14 * 4字节），这似乎可以接受。

在64位计算机上，我们使用3级基数树。



## 释放对象

当一个对象被释放时，我们计算它的页码并在中央数组中查找以找到相应的span对象。span告诉我们对象是否小，并告知其所在空闲列表类对象size 是否小。如果对象较小，则将其插入当前线程的线程缓存中的相应空闲列表中。如果线程缓存现在超过预定大小（默认为2MB），我们将运行垃圾回收器，将垃圾缓存中未使用的对象移到中央空闲列表中。

如果对象很大，则span会告诉我们该对象覆盖的页面范围。假设此范围是`[p,q]`。我们还将查找页面`p-1`和的`q+1`的span。如果这些相邻跨度中的任意一个是空闲的，则将它们与`[p,q]`span合并 。将结果span插入到页面堆中适当的空闲列表中。



## 中央空闲列表的小对象

如前所述，我们为每个尺寸级别保留一个中央空闲列表。每个中央空闲列表都组织为两级数据结构：一组span，以及每个span的空闲对象的链表。

通过从某个范围的链接列表中删除第一个条目，可以从中央空闲列表中分配一个对象。（如果所有跨度都有空的链表，则首先从中央页面堆中分配一个适当大小的跨度。）

通过将对象添加到其包含范围的链接列表中，可以将其返回到中央空闲列表。如果链接列表的长度现在等于该范围中小对象的总数，则该范围现在完全空闲，并返回到页面堆。



## 线程缓存的垃圾收集

从线程缓存收集垃圾对象可以控制缓存的大小，并将未使用的对象返回到中央空闲列表。一些线程需要较大的缓存才能正常运行，而其他线程则需要很少或根本不需要缓存。当线程缓存超过`max_size`时，垃圾收集将启动，然后该线程与其他线程竞争更大的缓存。

垃圾回收仅在释放对象期间运行。我们遍历缓存中的所有空闲列表，然后将一些对象从空闲列表移到相应的中央列表。

使用每个列表的低水位标记确定要从空闲列表中移动的对象的数量`L`。 `L` 记录自上次垃圾回收以来列表的最小长度。请注意，我们可以在上一个垃圾回收时通过`L`个对象将列表缩短， 而无需对中央列表进行任何额外的访问。我们将过去的历史用作将来访问的预测，并将`L/2` 对象从线程缓存空闲列表移动到相应的中央空闲列表。该算法具有很好的属性，即如果某个线程停止使用特定大小，则该大小的所有对象将迅速从线程缓存移到中央空闲列表，其他线程可以在该列表中使用它们。

如果一个线程持续释放的某个特定大小的对象多于其分配的数量，则此`L/2`行为将至少导致 `L/2`对象始终位于空闲列表中。为了避免以这种方式浪费内存，我们缩小了要聚合的[空闲列表](https://gperftools.github.io/gperftools/tcmalloc.html#Sizing_Thread_Cache_Free_Lists)的最大长度`num_objects_to_move`（另请参见 [调整线程缓存空闲列表的大小](https://gperftools.github.io/gperftools/tcmalloc.html#Sizing_Thread_Cache_Free_Lists)）。

```
垃圾回收
  if（L！= 0 && max_length> num_objects_to_move）{ 
    max_length = max（max_length-num_objects_to_move，num_objects_to_move）
  }
```

线程缓存超过了`max_size`表明该线程将从更大的缓存中受益。简单地增加`max_size`将在具有许多活动线程的程序中使用过多的内存。开发人员可以使用--tcmalloc_max_total_thread_cache_bytes标志来绑定使用的内存。

每个线程缓存都以一个较小的`max_size` （例如64KB）开头，以便空闲线程不会预分配它们不需要的内存。每次缓存运行垃圾回收时，它也会尝试扩大其`max_size`。如果线程缓存大小的总和小于--tcmalloc_max_total_thread_cache_bytes，则 `max_size`很容易增长。如果不是，线程缓存1将尝试通过减少线程缓存2的max_size来从线程缓存2窃取（选择循环）`max_size`。这样，活动性更高的线程将会更多的窃取其他线程的内存，而不是自己的内存中。大多数情况下，空闲线程以小缓存结束，而活动线程以大缓存结束。请注意，这种窃取可能导致线程缓存大小的总和大于--tcmalloc_max_total_thread_cache_bytes，直到线程缓存2释放一些内存以触发垃圾回收为止。



## 性能说明

### PTMalloc2单元测试

PTMalloc2软件包（现在是glibc的一部分）包含一个unittest程序`t-test1.c`。这会派生多个线程，并在每个线程中执行一系列分配和释放。线程仅通过内存分配器中的同步进行通信。

`t-test1`（包含在 `tests/tcmalloc/`中，并编译为 `ptmalloc_unittest1`）使用不同数量的线程（1-20）和最大分配大小（64字节-32KB）运行。这些测试使用RedHat 9的Linux glibc-2.3.2在启用超线程的2.4GHz双Xeon系统上运行，每个测试中每个线程执行一百万次操作。在每种情况下，测试均正常运行一次，并使用 `LD_PRELOAD=libtcmalloc.so`。

下图显示了针对几种不同指标的TCMalloc与PTMalloc2的性能。首先，对于不同数量的线程，每秒经过的总操作数（百万）与最大分配大小之比。可用于生成这些图的原始数据（`time`实用程序的输出 ） `t-test1.times.txt`。

| ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.1.threads.png)  | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.2.threads.png)  | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.3.threads.png)  |
| ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.4.threads.png)  | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.5.threads.png)  | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.8.threads.png)  |
| ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.12.threads.png) | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.16.threads.png) | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspersec.vs.size.20.threads.png) |

- TCMalloc比PTMalloc2具有更一致的可扩展性-对于所有线程数大于1的情况，小分配可达到约7-9百万个操作/秒，大分配可降低至约200万个操作/秒。单线程的情况显然是异常的，因为它只能使单个处理器保持繁忙，因此可以实现更少的运算/秒。PTMalloc2的每秒操作数差异更大-对于较小的分配，峰值约为400万操作/秒，对于较大的分配，则降至<100万操作/秒。
- 在大多数情况下，尤其是对于小型分配，TCMalloc比PTMalloc2快。在TCMalloc中，线程之间的争用问题不大。
- 随着分配大小的增加，TCMalloc的性能会下降。这是因为当每个线程缓存达到阈值（默认为2MB）时，将对其进行垃圾回收。使用较大的分配大小，在垃圾被垃圾回收之前，可以将较少的对象存储在缓存中。
- 最大分配大小为〜32K时，TCMalloc的性能明显下降；较大尺寸时，性能下降的速度较慢。这是由于每个线程缓存中对象的最大大小为32K。对于大于此TCMalloc的对象，从中央页堆分配。



接下来，CPU的每秒操作数（百万）与线程数的关系（最大分配大小为64字节-128 KB）。

| ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.64.bytes.png)    | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.256.bytes.png)   | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.1024.bytes.png)   |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.4096.bytes.png)  | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.8192.bytes.png)  | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.16384.bytes.png)  |
| ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.32768.bytes.png) | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.65536.bytes.png) | ![img](https://gperftools.github.io/gperftools/tcmalloc-opspercpusec.vs.threads.131072.bytes.png) |

在这里，我们再次看到，TCMalloc比PTMalloc2更一致，更高效。对于最大分配大小小于32K的情况，TCMalloc通常使用大量线程来实现每秒约2-250万个操作，而PTMalloc通常可以实现每秒0.5-1百万个操作，在很多情况下可以实现很多小于这个数字。超过32K最大分配大小后，TCMalloc的CPU时间下降到每秒1-150万，而对于大量线程，PTMalloc的下降几乎为零（即，使用PTMalloc时，大量的CPU时间被消耗掉，等待大量的锁定多线程情况）。



### 修改运行时行为

您可以通过环境变量更好地控制tcmalloc的行为。

一般有用的标志：

| `TCMALLOC_SAMPLE_PARAMETER`             | 默认值：0          | 采样动作之间的近似差距。也就是说，我们大约每`tcmalloc_sample_parmeter`分配一个字节取一个样本 。可通过`MallocExtension::GetHeapSample()`或 获取此采样堆信息 `MallocExtension::ReadStackTraces()`。合理的值为524288。                                                                                                                                                                                                                   |
| --------------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TCMALLOC_RELEASE_RATE`                 | 默认值：1.0        | 在`madvise(MADV_DONTNEED)`支持它的系统上，通过释放未使用的内存到系统的速率 。零表示我们永远不会将内存释放回系统。增加此标志以更快地返回内存；减少它以更慢地返回内存。合理的比率在[0,10]范围内。                                                                                                                                                                                                                                       |
| `TCMALLOC_LARGE_ALLOC_REPORT_THRESHOLD` | 默认值：1073741824 | 大于此值的分配将导致堆栈跟踪转储到stderr。每次我们打印一条消息时，转储堆栈跟踪的阈值将增加1.125倍，以便每60条消息，阈值将自动增加约1000倍。这限制了此标志生成的额外日志记录的数量。该标志的默认值非常大，因此除非覆盖了该标志，否则您将看不到任何额外的日志记录。                                                                                                                                                                     |
| `TCMALLOC_MAX_TOTAL_THREAD_CACHE_BYTES` | 默认值：16777216   | 约束分配给线程缓存的字节总数。此限制不是严格的，因此在某些情况下缓存可能会超过此限制。该值默认为16MB。对于具有多个线程的应用程序，这可能不是足够大的缓存，这可能会影响性能。如果您怀疑由于TCMalloc中的锁争用而导致应用程序无法扩展到多个线程，则可以尝试增加此值。这可能会提高性能，但以TCMalloc占用额外的内存为代价。有关更多详细信息，请参见[ 垃圾回收](https://gperftools.github.io/gperftools/tcmalloc.html#Garbage_Collection)。 |

高级“ tweaking”标志，可更精确地控制tcmalloc如何尝试从内核分配内存。

| `TCMALLOC_SKIP_MMAP`              | 默认值：false       | 如果为true，请不要尝试用于`mmap`从内核获取内存。                                        |
| --------------------------------- | ------------------- | --------------------------------------------------------------------------------------- |
| `TCMALLOC_SKIP_SBRK`              | 默认值：false       | 如果为true，请不要尝试用于`sbrk`从内核获取内存。                                        |
| `TCMALLOC_DEVMEM_START`           | 默认值：0           | 用于`/dev/mem` 分配的物理内存起始位置（以MB为单位）。将此设置为0将禁用`/dev/mem` 分配。 |
| `TCMALLOC_DEVMEM_LIMIT`           | 默认值：0           | 用于`/dev/mem` 分配的物理内存限制位置（以MB为单位）。将此设置为0表示没有限制。          |
| `TCMALLOC_DEVMEM_DEVICE`          | 默认值：/ dev / mem | 用于分配非托管内存的设备。                                                              |
| `TCMALLOC_MEMFS_MALLOC_PATH`      | 默认值：“”          | 如果已设置，请指定安装了ugettlbfs或tmpfs的路径。这可以允许更快的分配。                  |
| `TCMALLOC_MEMFS_LIMIT_MB`         | 默认值：0           | 将总memfs分配大小限制为指定的MB数。0表示“无限制”。                                      |
| `TCMALLOC_MEMFS_ABORT_ON_FAIL`    | 默认值：false       | 如果为true，则每当memfs_malloc未能满足分配时，则中止（）。                              |
| `TCMALLOC_MEMFS_IGNORE_MMAP_FAIL` | 默认值：false       | 如果为true，则忽略mmap的失败。                                                          |
| `TCMALLOC_MEMFS_MAP_PRIVATE`      | 默认值：false       | 如果为true，则通过memfs而不是MAP_SHARED进行映射时，请使用MAP_PRIVATE。                  |

## 修改代码中的行为

``malloc_extension.h`中的`MallocExtension`类 提供了一些旋钮，您可以在程序中对其进行调整，以影响tcmalloc的行为。

### 将内存释放回系统

默认情况下，tcmalloc会随着时间的推移逐渐将不再使用的内存释放回内核。该[tcmalloc_release_rate](https://gperftools.github.io/gperftools/tcmalloc.html#runtime)标志控制如何快速发生这种情况。您还可以在程序执行中的给定时间点强制释放，如下所示：

```
   MallocExtension :: instance（）-> ReleaseFreeMemory（）;
```

您还可以调用在运行时`SetMemoryReleaseRate()`更改 `tcmalloc_release_rate`值，或 `GetMemoryReleaseRate`查看当前的释放速率。

### 内存自省

有几种例程可以使人理解形式的当前内存使用情况：

```
   MallocExtension :: instance（）-> GetStats（buffer，buffer_length）; 
   MallocExtension :: instance（）-> GetHeapSample（＆string）; 
   MallocExtension :: instance（）-> GetHeapGrowthStacks（＆string）;
```

最后两个创建文件的格式与heap-profiler相同，并且可以作为数据文件传递给pprof。第一个是人类可读的，用于调试。

### 通用Tcmalloc状态

TCMalloc支持设置和检索任意“属性”：

```
   MallocExtension :: instance（）-> SetNumericProperty（property_name，value）; 
   MallocExtension :: instance（）-> GetNumericProperty（property_name，＆value）;
```

应用程序可以设置和获取这些属性，但是最有用的是库设置属性时，应用程序可以读取它们。这是TCMalloc定义的属性；您可以通过以下调用 `MallocExtension::instance()->GetNumericProperty("generic.heap_size", &value);`：

| `generic.current_allocated_bytes`           | 应用程序使用的字节数。这通常与操作系统报告的内存使用情况不匹配，因为它不包括TCMalloc开销或内存碎片。                                                                                                                                |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `generic.heap_size`                         | TCMalloc保留的系统内存字节。                                                                                                                                                                                                        |
| `tcmalloc.pageheap_free_bytes`              | 页面堆中空闲的映射页面中的字节数。这些字节可用于满足分配请求。它们始终计入虚拟内存使用率，并且除非操作系统将基础内存换出，否则它们也计入物理内存使用率。                                                                            |
| `tcmalloc.pageheap_unmapped_bytes`          | 页面堆中未映射的空闲页面中的字节数。这些字节可能已经通过MallocExtension“释放”调用之一释放回了操作系统。它们可用于满足分配请求，但通常会导致页面错误。它们始终计入虚拟内存使用情况，并且取决于操作系统，通常不计入物理内存使用情况。 |
| `tcmalloc.slack_bytes`                      | pageheap_free_bytes和pageheap_unmapped_bytes的总和。仅提供向后兼容性。不使用。                                                                                                                                                      |
| `tcmalloc.max_total_thread_cache_bytes`     | TCMalloc为小对象专用的内存量有一个限制。在某些情况下，更高的数字会折衷更多的内存使用量，从而提高效率。                                                                                                                              |
| `tcmalloc.current_total_thread_cache_bytes` | TCMalloc使用的某些内存的度量（用于小对象）。                                                                                                                                                                                        |

## 注意事项

对于某些系统，TCMalloc可能无法与未链接的应用程序`libpthread.so`（或OS上的等效文件）无法正常工作。它应该在使用glibc 2.3的Linux上运行，但是尚未测试其他OS / libc组合。

TCMalloc可能比其他malloc占用更多的内存（但往往没有其他malloc可能发生的大量崩溃）。特别是，在启动时，TCMalloc分配大约240KB的内部存储器。

不要尝试将TCMalloc加载到正在运行的二进制文件中（例如，在Java程序中使用JNI）。二进制文件将使用系统malloc分配一些对象，并可能尝试将它们传递给TCMalloc进行释放。TCMalloc将无法处理此类对象。

------



