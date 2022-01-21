# ZGC 概览

<!-- TOC -->

- [ZGC 概览](#zgc-概览)
    - [Features](#features)
        - [不足](#不足)
    - [支持的平台](#支持的平台)
    - [配置和调优](#配置和调优)
        - [常见参数](#常见参数)
    - [启用ZGC](#启用zgc)
    - [设置堆大小](#设置堆大小)
    - [设置并发 GC 线程](#设置并发-gc-线程)
    - [将未使用的内存返回给操作系统](#将未使用的内存返回给操作系统)
    - [在 Linux 上启用大页面](#在-linux-上启用大页面)
    - [在Linux上启用透明的巨大页面](#在linux上启用透明的巨大页面)
    - [启用NUMA支持](#启用numa支持)
    - [启用 GC 日志记录](#启用-gc-日志记录)
    - [迭代日志](#迭代日志)
        - [JDK 17](#jdk-17)
    - [JDK 16](#jdk-16)
    - [JDK 15](#jdk-15)
    - [JDK 14](#jdk-14)
    - [JDK 13](#jdk-13)
        - [JDK 12](#jdk-12)
        - [JDK 11](#jdk-11)
    - [FAQ](#faq)
        - [ZGC中的 "Z "代表什么？](#zgc中的-z-代表什么)
        - [它的发音是 "zed gee see "还是 "zee gee see"？](#它的发音是-zed-gee-see-还是-zee-gee-see)
    - [参考文献](#参考文献)

<!-- /TOC -->


Z Garbage Collector，也称为ZGC，是一种可扩展的低延迟垃圾收集器，旨在满足以下目标：

- 最大暂停时间在亚毫秒级
- 暂停时间不会随着堆、live-set 或 root-set 的大小而增加
- 处理大小从8MB到16TB的堆
- ZGC 最初是作为 JDK 11 中的一项实验性功能引入的，并在 JDK 15 中被宣布为Production Ready。

ZGC 的核心是一个并发垃圾收集器，这意味着所有繁重的工作都在Java 线程继续执行的同时完成。这极大地限制了垃圾收集对应用程序响应时间的影响。

这个OpenJDK项目由HotSpot Group赞助。

## Features

- 不分代的垃圾回收器，即垃圾回收时对全量内存进行标记，但是回收时仅针对分内存回收，优先回收垃圾比较多的页面。
- 仅支持Linux64位系统，不支持32位平台。
- 不支持使用压缩指针。
- 内存分区管理，且支持不同的分区粒度，在ZGC中分区称为页面（page),有小页面、中页面、大页面3种。
- 具有颜色指针（color pointer),通过设计不同的标记位区分不同的虚拟空间，而这些不同标记位指示的不同虚拟空间通过mmap映射在同一物理地址；颜色指针能够快速实现并发标记、转移和重定位。
- 设计了读屏障，实现了并发标记和并发转移的处理。
- 支持NUMA，尽量把对象分配在访问速度比较快的地方。

### 不足

- 仅实现了单代内存管理，也就是说没有考虑热点数据与冷数据，分代内存管理在C4中已经得到支持。据Azul官网文章介绍，所实现的分代的内存管理器比没有分代的内存管理器效率高10倍，也就是说ZGC还有巨大的进步空间。
- C2的支持还不够完善。
- 不支持Graal、HDSB等功能。
- 一些功能尚待完善，比如尚不支持类回收。
- 稳定性尚需提高。



##  支持的平台


|  platform   | Since  |
|  ----  | ----  |
| Linux/x64	| 	JDK 15（自 JDK 11 起是实验性的） 	|  
|  Linux/AArch64	| 	JDK 15（自 JDK 13 起实验性） 	|  
|  Linux/PPC	| 	JDK 17	 |  
|  macOS/x64	| 	JDK 15（自 JDK 14 以来的实验性版|  本）|  
|  macOS/AArch64	| 	JDK 17	 |  
|  Windows/x64	| 	JDK 15（自 JDK 14 以来的实验性版|  本） |  
|  Windows/AArch64	| 	JDK 16	 |  
 

## 配置和调优

### 常见参数

**通用的GC 选项**      
- -XX:MinHeapSize, -Xms    
- -XX:InitialHeapSize, -Xms |-XX:ZCollectionInterval
- -XX:MaxHeapSize, -Xmx |-XX:ZFragmentationLimit
- -XX:SoftMaxHeapSize 
- -XX:ConcGCThreads
- -XX:ParallelGCThreads
- -XX:UseDynamicNumberOfGCThreads
- -XX:UseLargePages
- -XX:UseTransparentHugePages
- -XX:UseNUMA
- -XX:SoftRefLRUPolicyMSPerMB
- -XX:AllocateHeapAt

**ZGC选项**

- -XX:ZAllocationSpikeTolerance
- -XX:ZAllocationSpikeTolerance   
- -XX:ZMarkStackSpaceLimit
- -XX:ZProactive
- -XX:ZUncommit
- -XX:ZUncommitDelay

**ZGC 诊断选项（-XX:+UnlockDiagnosticVMOptions)**
- -XX:ZStatisticsInterval
- -XX:ZVerifyForwarding
- -XX:ZVerifyMarking
- -XX:ZVerifyObjects
- -XX:ZVerifyRoots
- -XX:ZVerifyViews

## 启用ZGC

-XX:+UseZGC

## 设置堆大小

ZGC 最重要的调优选项是设置最大堆大小 ( -Xmx<size>)。由于 ZGC 是并发收集器，因此必须选择最大堆大小，以便 
- 堆可以容纳应用程序的实时数据集
- 堆中有足够的空间来允许在 GC 进行时处理分配程序。

需要多大的余量在很大程度上取决于应用程序的分配率和实时集的大小。一般来说，你给ZGC的内存越多越好。但同时，浪费内存也是不可取的，所以关键是要在内存使用和GC需要运行的频率之间找到一个平衡。

## 设置并发 GC 线程

第二个调整选项是设置并发的GC线程数量（-XX:ConcGCThreads=<number>）。ZGC有启发式方法来自动选择这个数字。这种启发式方法通常工作得很好，但根据应用程序的特点，可能需要调整。这个选项本质上决定了应该给GC多少CPU时间。给它太多，GC会从应用中窃取过多的CPU时间。给得太少，应用程序分配垃圾的速度可能比GC收集垃圾的速度快。

**注意**！一般来说，如果低延迟（即低应用响应时间）对你的应用很重要，那么永远不要过度配置你的系统。理想情况下，你的系统的CPU利用率不应超过70%。

## 将未使用的内存返回给操作系统

默认情况下，ZGC不提交未使用的内存，并将其返回给操作系统。这对于那些需要考虑内存占用的应用程序和环境是很有用的。这个功能可以用-XX:-ZUncommit来禁用。此外，内存不会被取消提交，这样堆的大小就会缩小到最小堆大小（-Xms）以下。这意味着如果最小堆大小（-Xms）被配置为等于最大堆大小（-Xmx），这个功能将被隐式禁用。

可以用-XX:ZUncommitDelay=<seconds>（默认是300秒）来配置未提交延迟。这个延迟指定了内存在多长时间内未被使用才有资格被解密。

**注意** ！在Linux上，取消提交未使用的内存需要fallocate(2)支持FALLOC_FL_PUNCH_HOLE，它首次出现在内核版本3.5（针对tmpfs）和4.3（针对hugetlbfs）。


## 在 Linux 上启用大页面

将ZGC配置为使用大页面通常会产生更好的性能（在吞吐量、延迟和启动时间方面），并且没有真正的缺点，只是它的设置稍微复杂一些。设置过程通常需要root权限，这就是为什么它在默认情况下不启用。

在Linux/x86上，大页面（也被称为 "巨大页面"）的大小为2MB。

让我们假设你想要一个16G的Java堆。这意味着你需要16G / 2M = 8192个巨大的页面。

首先将至少16G（8192页）的内存分配给巨大页池。"至少 "这个部分很重要，因为在JVM中启用大页面意味着不仅GC会尝试将这些页面用于Java堆，而且JVM的其他部分也会尝试将其用于各种内部数据结构（代码堆、标记位图等）。因此，在这个例子中，我们将保留9216个页面（18G），允许2G的非Java堆分配使用大页面。

配置系统的巨大页面池，使其拥有所需的页面数量（需要root权限）。

```
echo 9216 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

```

注意，如果内核不能找到足够的空闲的巨大页面来满足请求，上述命令不保证成功。还要注意，内核可能需要一些时间来处理这个请求。在继续之前，检查分配给池子的巨大页面的数量，以确保请求是成功的并且已经完成。

```
$ cat /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
9216 
```

注意！如果你使用的是>=4.14的Linux内核，那么接下来的步骤（挂载hugetlbfs文件系统）可以跳过。然而，如果你使用的是旧内核，那么ZGC需要通过hugetlbfs文件系统来访问大的页面。

挂载一个hugetlbfs文件系统（需要root权限），并使运行JVM的用户能够访问它（在这个例子中，我们假设这个用户的uid是123）。

```
$ mkdir /hugepages
$ mount -t hugetlbfs -o uid=123 nodev /hugepages 
```

现在使用-XX:+UseLargePages选项启动JVM。

```
$ java -XX:+UseZGC -Xms16G -Xmx16G -XX:+UseLargePages ...

```

如果有多个可访问的hugetlbfs文件系统可用，那么（也只有这时）你还必须使用-XX:AllocateHeapAt来指定你想使用的文件系统的路径。例如，假设有多个可访问的hugetlbfs文件系统被挂载，但你特别想使用的文件系统被挂载在/hugepages，那么使用以下选项。

```
$ java -XX:+UseZGC -Xms16G -Xmx16G -XX:+UseLargePages -XX:AllocateHeapAt=/hugepages ...
```

**注意** ！除非采取适当的措施，否则巨大的页面池的配置和hugetlbfs文件系统的挂载在重启后是不持久的。


## 在Linux上启用透明的巨大页面


使用显式大页面（如上所述）的一个替代方法是使用透明的大页面。对于延迟敏感的应用，通常不推荐使用透明的大页面，因为它往往会导致不必要的延迟峰值。然而，这可能值得试验一下，看看你的工作负载是否/如何受到它的影响。但请注意，你的里程可能会有所不同。

注意！在Linux上，使用ZGC并启用透明的巨大页面需要内核>=4.7。

使用以下选项在虚拟机中启用透明的巨大页面。

```
-XX:+UseLargePages -XX:+UseTransparentHugePages
```

这些选项告诉JVM为其映射的内存发出madvise(..., MADV_HUGEPAGE)调用，这在madvise模式下使用透明的巨大页面时很有用。

为了启用透明的巨大页面，你还需要配置内核，启用madvise模式。

```
$ echo madvise > /sys/kernel/mm/transparent_hugepage/enabled

```
和
```
$ echo advise > /sys/kernel/mm/transparent_hugepage/shmem_enabled
```

## 启用NUMA支持

ZGC支持NUMA，这意味着它将尽力把Java堆分配到NUMA的本地内存。这个功能默认是启用的。然而，如果JVM检测到它只能使用单个NUMA节点的内存，它将自动被禁用。一般来说，你不需要担心这个设置，但如果你想明确地覆盖JVM的决定，你可以通过使用-XX:+UseNUMA或-XX:-UseNUMA选项来实现。

## 启用 GC 日志记录

GC日志是通过以下命令行选项启用的。

```
-Xlog:<tag set>,[<tag set>, ...]:<log file>
```
关于这个选项的一般信息/帮助。

```-Xlog:help```

启用基本的日志记录（每个GC有一行输出）。

```-Xlog:gc:gc.log```

启用对调整/性能分析有用的GC日志。

```-Xlog:gc*:gc.log```

其中gc*意味着记录所有包含gc标签的标签组合，而:gc.log意味着将日志写入一个名为gc.log的文件。

## 迭代日志

### JDK 17
- 动态的GC线程数量
- 减少了标记堆栈内存的使用
- 支持macOS/aarch64
- 暂停和循环的GarbageCollectorMXBeans
- 快速的JVM终止
## JDK 16
- 并发线程堆栈扫描（JEP 376）
- 支持原地重定位
- 性能改进（转发表的分配/初始化等）。
## JDK 15
- 生产就绪 (JEP 377)
- 改进了NUMA意识
- 改进了分配并发性
- 支持类数据共享（CDS）
- 支持将堆放在NVRAM上
- 支持压缩的类指针
- 支持增量不提交
- 修复了对透明的巨大页面的支持
- 额外的JFR事件
## JDK 14
- 支持macOS (JEP 364)
- 支持Windows（JEP 365）
- 支持小/小堆（低至8M）。
- 支持JFR泄漏分析器
- 支持有限和不连续的地址空间
- 并行预触摸（当使用-XX:+AlwaysPreTouch）。
- 性能改进（克隆本征，等等）。
- 稳定性改进
## JDK 13
- 将最大堆大小从4TB增加到16TB
- 支持不提交未使用的内存（JEP 351）。
- 支持 -XX:SoftMaxHeapSIze
- 对Linux/AArch64平台的支持
- 减少了安全时间（Time-To-Safepoint
### JDK 12
- 支持并发的类卸载
- 进一步减少了暂停时间
### JDK 11
- ZGC的初始版本
- 不支持类卸载（使用 -XX:+ClassUnloading 无效）。

## FAQ

### ZGC中的 "Z "代表什么？
它并不代表什么，ZGC只是一个名字。它最初是受ZFS（文件系统）的启发，或者说是对它的致敬，ZFS在很多方面都是革命性的。最初，ZFS是 "Zettabyte File System "的首字母缩写，但这个意思被放弃了，后来据说它不代表任何东西。它只是一个名字而已。


### 它的发音是 "zed gee see "还是 "zee gee see"？
没有首选的发音，两种都可以。

## 参考文献

- [OpenJDK Wiki Main](https://wiki.openjdk.java.net/display/zgc/Main)
- 《新一代垃圾回收器-ZGC设计与实现》-- 彭成寒