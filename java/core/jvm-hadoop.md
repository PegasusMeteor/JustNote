# HBase群集调整G1GC
<!-- TOC -->

- [HBase群集调整G1GC](#hbase%E7%BE%A4%E9%9B%86%E8%B0%83%E6%95%B4g1gc)
  - [HBase的G1GC](#hbase%E7%9A%84g1gc)

<!-- /TOC -->

前面我们介绍过，G1GC 是现在比较流行的JVM 垃圾回收。所以，我们就结合G1GC 来讨论下Hadoop的GC。

## HBase的G1GC

首先，我们本篇文章的主要依据就是官方的blog。[https://blogs.apache.org/hbase/entry/tuning_g1gc_for_your_hbase](https://blogs.apache.org/hbase/entry/tuning_g1gc_for_your_hbase)
