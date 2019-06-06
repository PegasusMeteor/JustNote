# HBase 调整G1GC
<!-- TOC -->

- [HBase 调整G1GC](#hbase-%E8%B0%83%E6%95%B4g1gc)
  - [HBase的G1GC](#hbase%E7%9A%84g1gc)
  - [参数配置](#%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE)
  - [如何调整HBase集群](#%E5%A6%82%E4%BD%95%E8%B0%83%E6%95%B4hbase%E9%9B%86%E7%BE%A4)
    - [开始之前: GC and HBase monitoring](#%E5%BC%80%E5%A7%8B%E4%B9%8B%E5%89%8D-gc-and-hbase-monitoring)
    - [Step 0：推荐默认值](#step-0%E6%8E%A8%E8%8D%90%E9%BB%98%E8%AE%A4%E5%80%BC)
    - [Step 1:确定预期的HBase最大使用率](#step-1%E7%A1%AE%E5%AE%9A%E9%A2%84%E6%9C%9F%E7%9A%84hbase%E6%9C%80%E5%A4%A7%E4%BD%BF%E7%94%A8%E7%8E%87)
    - [Step 2: 设置堆大小，IHOP和Eden大小](#step-2-%E8%AE%BE%E7%BD%AE%E5%A0%86%E5%A4%A7%E5%B0%8Fihop%E5%92%8Ceden%E5%A4%A7%E5%B0%8F)
    - [Step 3:根据使用上限调整HBase配置](#step-3%E6%A0%B9%E6%8D%AE%E4%BD%BF%E7%94%A8%E4%B8%8A%E9%99%90%E8%B0%83%E6%95%B4hbase%E9%85%8D%E7%BD%AE)
    - [Step 4: 配置其他建议的参数以提高GC的性能](#step-4-%E9%85%8D%E7%BD%AE%E5%85%B6%E4%BB%96%E5%BB%BA%E8%AE%AE%E7%9A%84%E5%8F%82%E6%95%B0%E4%BB%A5%E6%8F%90%E9%AB%98gc%E7%9A%84%E6%80%A7%E8%83%BD)
    - [Run it](#run-it)
  - [其他参考](#%E5%85%B6%E4%BB%96%E5%8F%82%E8%80%83)

<!-- /TOC -->

前面我们介绍过，G1GC 是现在比较流行的JVM 垃圾回收。所以，我们就结合G1GC 来讨论下Hadoop的GC。

## HBase的G1GC

首先，我们本篇文章的主要依据就是官方的blog。[https://blogs.apache.org/hbase/entry/tuning_g1gc_for_your_hbase](https://blogs.apache.org/hbase/entry/tuning_g1gc_for_your_hbase)

鉴于已经有官方blog作为资料，并且实验环境并不是很充足，所以我们直接叙述结论。

## 参数配置

首先看下，我们实际工作中的HBase的参数配置。以及各项参数的实际含义。

参数|含义
-|-
-XX:+UseG1GC| 显式地要求使用G1 GC
-Xms64G | 设置JVM最大可用内存为
-Xmx64G | 设置JVM最大可用内存为64G
-XX:PermSize=256m | JVM初始分配的非堆内存
-XX:G1NewSizePercent=4 | 新生代最小值，默认值5%
-XX:MaxGCPauseMillis=200 | 设置G1收集过程目标时间，默认值200ms，不是硬性条件
-XX:ParallelGCThreads=16 | STW期间，并行GC线程数
-XX:MaxTenuringThreshold=4 | 年龄阈值，默认15（对象被复制的次数）
-XX:+UnlockExperimentalVMOptions | 与+UseG1GC 一起使用,来解锁参数,应该是一种安全机制
-XX:+ParallelRefProcEnabled | 默认为false，并行的处理Reference对象，如WeakReference，除非在GC log里出现Reference处理时间较长的日志，否则效果不会很明显。
-XX:-ResizePLAB | 是否启动动态修改

## 如何调整HBase集群

### 开始之前: GC and HBase monitoring

可以使用任何工具跟踪收集  `block cache`, `memstore` 和 `static index size` 这些指标，可以从 `RegionServer JMX metrics` 中找到。  

- “memStoreSize”
- “blockCacheSize”
- “staticIndexSize”

还可以使用官方的 `collectd` 插件来跟踪一段时间内的G1GC性能,并使用 `gc_log_visualizer`来了解特定的GC日志。要使用这些内容就要在 RegionServers 上跟踪这些信息。

- -Xloggc:$GC_LOG_PATH
- -verbosegc
- -XX:+PrintGC
- -XX:+PrintGCDateStamps
- -XX:+PrintAdaptiveSizePolicy
- -XX:+PrintGCDetails
- -XX:+PrintGCApplicationStoppedTime
- -XX:+PrintTenuringDistribution

还建议使用某种GC日志轮换，例如:

- -XX：+ UseGCLogFileRotation
- -XX：NumberOfGCLogFiles = 5
- -XX：GCLogFileSize = 20M

### Step 0：推荐默认值

建议将以下JVM参数和值作为HBase RegionServers的默认值:

- -XX：+ UseG1GC
- -XX：+ UnlockExperimentalVMOptions
- -XX：MaxGCPauseMillis = 50
- -XX：-OmitStackTraceInFastThrow
- -XX：ParallelGCThreads = 8+（逻辑处理器-8）
- -XX：+ ParallelRefProcEnabled
- -XX：+ PerfDisableSharedMem
- -XX：-ResizePLAB

### Step 1:确定预期的HBase最大使用率

使用上面提到的RegionServer JMX指标，查找整个集群中每个指标的最大值

- Maximum block cache size.
- Maximum memstore size.
- Maximum static index size.

将每个最大值扩展到 110%，以适应最大使用值的轻微增幅。这是该指标的使用上限: 例如最大10 GB记录的memstore→11 GB memstore上限.

理想情况下，在过去一周或一个月内跟踪这些指标，就可以找到该时间段内的最大值。如果不是，在计算 memstore和static index size 上限时要比110％ 更高一些。尤其是 Memstore，随着时间的推移，可能会变的更大。

### Step 2: 设置堆大小，IHOP和Eden大小

先从比较低的 Eden 大小开始设置：如果不是很确定，8% 的堆的大小就可以。

```java
-XX:G1NewSizePercent = 8
```

- 增加Eden大小会增加单个GC pause 时间，但会略微减少GC中花费的总时间。
- 减小Eden大小会产生相反的效果：GC pause 时间越短，GC中的总体时间略长。

用Step 1中的Eden大小和使用上限确定必要的堆大小:

Heap ≥ (M + B + O + E) ÷ 0.7

- M = memstore cap, GB
- B = block cache cap, GB
- O = static index cap, GB
- E = Eden size, GB

根据计算的值设置固定堆大小的JVM参数,例如：

```java
-Xms40960m -Xmx40960m
```

根据使用上限和堆大小在JVM中设置IHOP:

- IHOP = (memstore cap’s % heap + block cache cap’s % heap + overhead cap’s % heap + 20)
- -XX:InitiatingHeapOccupancyPercent = IHOP

### Step 3:根据使用上限调整HBase配置

根据使用上限和总堆大小,在 HBase 配置中设置 `block cache` 上限 和 `memstore` 上限比率:

- hfile.block.cache.size → block cache cap ÷ heap size
- hbase.regionserver.global.memstore.size → memstore cap ÷ heap size

### Step 4: 配置其他建议的参数以提高GC的性能

-XX:MaxTenuringThreshold = 1
-XX:G1HeapWastePercent = 10
-XX:G1MixedGCCountTarget = 16
-XX:G1HeapRegionSize = #M  #必须是2的幂，在[1..32]范围内,理想情况下，＃是这样的：堆大小÷＃MB = 2048个region。

### Run it

应用这些配置，然后重启RegionServers，看看他们的表现。

- 可以如上所述调整Eden大小，以优化较短的单个GC或减少GC中的总时间,如果这样做，请确保Eden +IHOP≤90％。
- 如果HBase客户端可能有非常突发的流量，可以考虑在IHOP和Eden之外添加堆空间（例如，IHOP + Eden加起来高达80％.

## 其他参考

- [G1GC Fundamentals (HubSpot blog)](https://product.hubspot.com/blog/g1gc-fundamentals-lessons-from-taming-garbage-collection)
- [Understanding G1GC Logs (Oracle blog)](https://blogs.oracle.com/poonam/entry/understanding_g1_gc_logs)
- [Tuning HBase Garbage Collection (Intel blog)](https://blogs.oracle.com/poonam/understanding-g1-gc-logs)
