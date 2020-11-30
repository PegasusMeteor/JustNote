# G1GC Q&A

前面的文章中，我们援引了美团技术博客的文章，简单介绍和了解了G1GC。但是心中还是有很多疑问，所以这篇文章，我就将这些疑惑一一列举出来，并一一解答。争取做到对G1GC有一个深入的详细的了解。

G1 指的是 Garbage First,GC 在JVM 中有两个含义，内存的分配和针对已分配内存的回收。

## 参考

- 《JVM G1 源码分析和调优》-- 彭成寒编著
- [G1GC 参数调优](https://www.oracle.com/cn/technical-resources/articles/java/g1gc.html)
- [The Garbage First Garbage Collector](https://www.oracle.com/java/technologies/javase/hotspot-garbage-collection.html)

## G1 的基本概念

### 如何设置Heap Region的大小

**G1 中每个Region的的大小都是相同的，如何设置Heap Region的大小？设置HR的时候有哪些考虑**?

HR 的大小直接影响分配和回收的效率。HR过大，可以存放多个对象，分配效率高，但是回收效率低。过小，则分配效率低下。为了均衡二者，所以HR设置了一个上下限，分别是1MB,2MB,4MB,...,32MB,也就是2的指数次幂。默认情况下，整个堆空间有2048个HR(该值可以通过最小的堆分区大小计算出来)。

- G1HeapRegionSize 可以用来指定HR的大小，一般默认为0.
- 不指定的时候，由G1自行推断。

按照默认值来计算的话，G1可以管理的最大内存为 2048×32MB = 64G。假设 xms=32G,xmx=128G,那么计算过程如下：

- average_heap_size=(32GB+128GB)/2= 80GB 判断是否设置过分区大小，如果有就使用，没有则根据初始内存和最大分配内存，获得平均值
- region_size=max(80GB/2048,1m)= 40M  并根据HR的个数得到Region的大小,和Region的下限进行比较，取两者的最大值。
- region_size= 32M  对region_size 按2的幂次进行对齐，保证其落在上下限范围内

那这样计算出来的每个 HR 的大小就是32MB，则根据最小内存和最大内存的计算，HR个数的变化范围就是从1024 到4096个。

### G1 大对象不使用 新生代，直接进入老年代，那什么是大对象？

简单来说，就是region_size 的一半。

### 新生代大小如何设置？

新生代大小指的是新生代内存空间的大小。G1中还新增了两个参数G1MaxNewSizePercent 和G1NewSizePercent用于控制新生代的大小。 整体逻辑如下：

- 如果设置新生代最大值（MaxNewSize）和最小值（NewSize），可以根据这些计算出最大的分区和最小的分区，注意设置了Xmn等价与设置了MaxNewSize和NewSize，且 NewSize=MaxNewSize。
- 如果设置 最大值（MaxNewSize）和最小值（NewSize） ，又设置了NewRatio，则忽略NewRatio。
- 如果没有设置最大值（MaxNewSize）和最小值（NewSize），但是设置了NewRatio，则最大值和最小值相同 (= 整个heap/(NewRatio+1))
- 如果没有设置最大值（MaxNewSize）和最小值（NewSize），或者只设置了其中一个，那么G1将根据G1MaxNewSizePercent （默认60）和G1NewSizePercent（默认5）占整个堆空间的比例来计算最大最小值。

### 分配新的分区时，如何扩展，一次扩展多少？

G1 是自适应扩展内存的， 参数 -XX:GCTimeRatio 表示GC与应用的耗时时间比，默认为9.即G1 GC时间与总时间占比不超过10%时，不需要动态扩展，当GC超过这个值时，可以动态扩展。计算方式是 100 × （1.0 /(1.0 + GCTimeRatio）。扩展时有一个参数，G1ExpandByPercentOfAvailable(默认20),即每次都从未提交的内存中申请20%。

### G1 停顿预测模型

G1 是一个响应时间优先的GC 算法。所以用户可以设定自己期望的响应时间来让G1自己进行模型计算。有一个参数MaxGCPauseMills，默认200ms。G1 会努力在这个期望值内完成GC，但是并不绝对。

G1 为了满足用户期望，就会根据历史数据进行一个模型预测，从而尽量满足用户设定的目标停顿时间。G1的预测逻辑是基于衰减平均值和衰减标准差。

### CardTable And RSet(Remember Set)

CardTable 在之前的GC中就已经存在了，它存在的目的是为了对内存的引用关系做标记。从而根据引用关系遍历活跃对象。
关于两者的介绍可以参看前面的文章。

### 参数介绍和调优

- G1HeapRegionSize 指定堆分区的大小，分区大小可以指定，也可以不指定，不指定时由内存管理器自动推断堆分区大小。
- xms/xmx 指定堆空间最小/最大值
- GCTimeRatio 指的是GC时间与整体时间的占比。前面我们已经介绍过计算，增大该值，能够减少GC占用，但是后果就是动态扩展内存更容易发生。
- G1NewSizePercent是一个实验参数。需要使用-XX:+UnlockExperimentalVMOptions 配合才能改变选项。有实验表明G1在回收Eden分区的时候，大概每GB需要100ms，所以可以根据停顿时间进行相应的调整。这个值在内存比较多大的时候，可以相应的减少。

注意： G1 中不需要设置MaxNewSize、NewSize、Xmn和NewRatio，原因是，一，G1对内存的管理是不连续的，即使重新分配一个堆分区代价也不高，G1的目标满足 垃圾收集停顿，如果设置了固定分区，则G1不能调整新生代的大小，可能就满足不了垃圾收集停顿了。

## G1 的对象分配
  
  G1 提供了两种对象分配策略： 基于线程本地分配缓冲区（Thread Local Allocation Buffer，TLAB）的快速分配和慢速分配。
无论快速分配还是慢速分配，都应该在STW（Stop the World）之外进行调用。

在分配线程对象时，从JVM 堆中分配一个固定大小的内存区域将于作为线程的私有缓冲区，这个缓冲区就是TLAB。只有为线程分配TLAB时候才需要锁定JVM。也就是加锁。其实这个比较好理解，在Java中线程是资源分配的最小单位。不同线程不共享TLAB.

当我们需要去分配一个对象时，优先从当前线程的TLAB去分配，不需要全局锁，因此就达到了快速分配的目的。

当不能进行快速分配时，就会进入慢速分配，而且慢的过程中会有一些 诸如大对象直接分配到老年代，分配不会收需要先进行GC等，失败一定次数之后，则分配失败。

### G1垃圾回收的时机

- 分配时发生回收，快速分配和慢速分配时都有可能存在内存不足，都有可能发生回收，回收之后再继续分配。
- 外部调用的回收，例如显式地调用了system.gc 。 JNI（Java Native Interface） 代码进入了临界区(synchronized)，为了保证安全需要加锁，加锁又发出了GC请求，导致GC等。

### 参数介绍和调优

- 在优化调试TLAB的时候，在调试环境可以打开PrintTLAB来观察TLAB的分配和使用情况
- UseTLAB，指是否打开TLAB，大量的实验证明使用TLAB能够加速TLAB的分配和使用的情况
- ResizeTLAB，是否允许动态调整。基准测试表明，使用动态调整TLAB大小，效率更高
- TLABSize，指设置TLAB的大小，实际使用中不要设置，设置之后就不能动态调整了。

## 新生代回收

内存分配时，剩余空间不能满足分配要求就会优先触发新生代回收（Young GC ，YGC）。

为了便于理解，下面用自己的话进行整理描述：

- 收集之前先STW.
- 选择要收集的区域，对于YGC来说，要收集的就是整个新生代。
- 进行并行任务处理。
- 将符合条件的对象复制到新的Survivor区,对象的field入栈等待复制处理.
- 处理老年代到新生代的代际引用，即更新RSet（前面的文章中有介绍）
- 对栈中的对象进行深度递归遍历复制对象。

### 参数介绍和调优

- ParallelGCThreads ,默认值为0，表示并行执行GC的线程个数。G1可以根据CPU的核数自动推断线程数。GC是CPU密集型，通常来说，线程个数不应该超过CPU核数，一般不用设置该值。
- ResizePLAB，默认为true。表示在垃圾回收之后会根据内存的使用情况来调整PLAB（promotion local allocation buffer）的大小，在一些测试中发现关闭这个参数可能有更好的效果。
- SurvivorRatio，默认值为8，指Eden和一个Survivor分区之间的比例。默认8:1:1。G1 中并会因为增大这个值，就导致Eden变小，因为Eden时根据GC的时间来预测的。

## 混合回收

混合回收可以总结为两个阶段：

- 第一阶段： 并发标记，目的就是识别老年代分区中的活跃对象，并计算分区中垃圾对象所占空间的多少，用于垃圾回收过程中判断是否回收分区
  - 初始标记子阶段
  - 并发标记子阶段
  - 再标记子阶段
  - 清理阶段
- 第二阶段： 垃圾回收。这个过程和新生代回收的步骤完全一致，重用了新生代的逻辑。最大的区别就是，不仅要回收新生代，还要回收并发标记中识别到的垃圾多的老年代分区。

下面就根据混合回收发生的逻辑顺序依次介绍一下这些阶段：

#### 初始标记子阶段

标记所有的根对象（根对象，全局对象，JNI对象）,根是对象图的起点，需要STW.混合回收的初始标记与YGC的初始标记几乎一样。实际上就是借用了YGC之后的结果，即Survivor分区作为根，所以混合回收一定发生再YGC之后，且不需要再一次进行初始标记。

#### 并发标记子阶段

当YGC执行结束之后，如果发现满足并发标记的条件，并发线程就开始并发标记。根据新生代的Survivor分区以及老年代的Rset开始并发标记。并发标记会对所有的分区进行标记，这个阶段并不需要STW,这时标记线程和应用程序线程同时运行。

#### 再标记子阶段

这是最后一个标记阶段。这时，G1需要暂停一下，找出所有未被访问到的对象，同时完成存活内存数据计算。

这个阶段也是并行执行的，通过参数 -XX:ParallelGCThreads 可以设置GC 暂停时可用端的GC 线程数。

#### 清理子阶段

再标记之后进入清理子阶段，也是需要STW的。清理子阶段需要完成虾米那的操作。

- 统计存活对象，主要是利用RSet和位图来实现。
- 交换标记位图，并为下次并发标记做准备
- 重置RSet，此时老年代已经标记完，如果标记后的分区没有引用对象，这说明引用已经发生改变，可以删除原来RSet里面的引用关系。
- 把空闲分区放到空闲分区列表中，这里的空闲指的是全部是垃圾对象的分区，如果分区还有任何分区活跃对象都不会被释放，真正释放是在混合GC中。

这个阶段容易让人误解，清理阶段并没有真正的GC，也不会执行存活对象的拷贝，极端情况下，该阶段结束之后，空闲分区列表和JVM内存的使用情况可能毫无变化。

#### 混合回收阶段

这个阶段是辉进行真正的GC的。 与YGC 一样，第一个步骤是从分区中选出若干个进行回收，这些被选中的分区称为Collect Set（简称CSet）；第二个步骤是把存活的对象复制到空闲的分区中区，同时把这些已经被回收的分区放到空闲分区列表中。垃圾回收总是要在一次新的YGC开始才会发生。

### 并发标记法---三色标记法

三色标记法是一个逻辑上的抽象

- 白色，没有被收集器表标记的对象
- 灰色，自身被标记到，但是其拥有的field字段引用到其他对象还没有被处理。
- 黑色, 自己本身被标记，同时field引用的对象也被标记。

这里虽然讲的很简单，但是实际实现过程中却非常复杂，这里先略过了。

### 参数介绍和调优

- InitiationHeapOccupancyPercent(IHOP),默认值是45，这个值是启用并发标记的先决条件。只有当老年代内存占总空间45%之后才会启动并发标记任务。增加该值，可能导致 并发标记花费更多时间。也会导致YGC或者MixedGC收集时的分区变少，还有可能导致FullGC。根据经验，这个值通常根据整体应用占用的平均值来设置，比平均内存稍微高一点此时性能最好（即YGC 和 MixedGC 比较快，FGC比较少），IHOP的设置非常有用，但是设置IHOP并不容易，需要不断尝试。
- G1ReservePercent,默认是10，当发现GC晋升失败导致FGC，可以增大这个值。
- HeapSizePerGCThread，默认为64M，可以简单地理解为每64M分配一个线程。

## FULL GC

DK 10 之前的FGC都是串行GC,但是不管是串行还是并行，过程和步骤都是一样的。而且采用的标记清除算法。

- 标记活跃对象
- 计算新对象的地址
- 把所有的引用都更新到新的地址上面
- 移动对象完成压缩

### 参数介绍和调优

- MinHeapFreeRatio，用于判断是否可以扩展堆空间。增大该值扩展概率变小，减少该值扩展几率变大
- MaxHeapFreeRatio，用于判断是否可以收缩堆空间。增大该值收缩概率变小，减少该值收缩几率变大

## 实践

找到一台运行Java程序的机器，运行`jps`命令,找到相应的进程ID，然后运行 `jmap -heap pid`就可以查看到该进程的堆栈以及GC信息。

```shell
jmap -heap 15784
Picked up JAVA_TOOL_OPTIONS: -Duser.timezone=UTC -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
Attaching to process ID 15784, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13

using thread-local object allocation.
Garbage-First (G1) GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 5368709120 (5120.0MB)
   NewSize                  = 1363144 (1.2999954223632812MB)
   MaxNewSize               = 3221225472 (3072.0MB)
   OldSize                  = 5452592 (5.1999969482421875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 2097152 (2.0MB)

Heap Usage:
G1 Heap:
   regions  = 2560
   capacity = 5368709120 (5120.0MB)
   used     = 1002454008 (956.0146408081055MB)
   free     = 4366255112 (4163.9853591918945MB)
   18.67216095328331% used
G1 Young Generation:
Eden Space:
   regions  = 436
   capacity = 3353346048 (3198.0MB)
   used     = 914358272 (872.0MB)
   free     = 2438987776 (2326.0MB)
   27.267041901188243% used
Survivor Space:
   regions  = 14
   capacity = 29360128 (28.0MB)
   used     = 29360128 (28.0MB)
   free     = 0 (0.0MB)
   100.0% used
G1 Old Generation:
   regions  = 29
   capacity = 1986002944 (1894.0MB)
   used     = 58735608 (56.01464080810547MB)
   free     = 1927267336 (1837.9853591918945MB)
   2.9574783953593173% used
```
