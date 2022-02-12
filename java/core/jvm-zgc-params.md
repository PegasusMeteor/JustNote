# ZGC 参数调优

<!-- TOC -->

- [ZGC 参数调优](#zgc-参数调优)
    - [ZGC 新引入的参数](#zgc-新引入的参数)

<!-- /TOC -->



## ZGC 新引入的参数

| 类型     | 参数                            | 默认值 | 描述  |
| -- | --- | -- | ----- |
| 生产选项 | ZPath                           | null   | 指定堆空间存储使用的文件系统，仅支持tmpfs或者hugelbfs                                                                                 |
| 生产选项 | ZAllocationSpikeTolerance       | 2      | 垃圾回收分配速率触发的修正预测参数，该参数越大，垃圾回收执行得越频繁                                                                  |
| 生产选项 | ZFragmentationLimit             | 25     | "指定页面中在垃圾回收期间允许存在的垃圾的最大比例。在垃圾回收期间，当页面中的垃圾空间超过该比例，则页面会进入回收，否则页面不会被回收 |
| "        |                                 |        |                                                                                                                                       |
| 生产选项 | ZStallOnOutOfMemory             | TRUE   | 指定JVM在内存不足时等待垃圾回收完成，而不是抛出OOM                                                                                    |
| 生产选项 | ZMarkStacksMax                  | 8GB    | 标记栈的最大空间，标记过程中如果标记栈的空间达到该值，则JVM会直接退出                                                                 |
| 生产选项 | ZCollectionInterval             | 0      | 允许自定义执行垃圾回收的间隔，0表示不执行自定义触发的规则                                                                             |
| 生产选项 | ZStatisticsInterval             | 10     | 输出统计信息的时间间隔                                                                                                                |
| 诊断选项 | ZStatisticsForceTrace           | FALSE  | 统计信息详情                                                                                                                          |
| 诊断选项 | ZProactive                      | TRUE   | 主动触发垃圾回收                                                                                                                      |
| 诊断选项 | ZUnmapBadViews                  | FALSE  | 仅把当前有效的地址映射到地址视图中                                                                                                    |
| 诊断选项 | ZVerifyMarking                  | FALSE  | 在标记开始和标记结束时验证标记栈，应该为空                                                                                            |
| 诊断选项 | ZVerifyForwarding               | FALSE  | 在并发转移中验证转移表（forwarding)没有重复的对象                                                                                     |
| 诊断选项 | ZSymbolTableUnloading           | FALSE  | 在第3步中是否卸载未使用的符号表                                                                                                       |
| 诊断选项 | ZWeakRoots                      | TRUE   | 弱根集合回收，为true时在垃圾回收周期的第3步中回收，为false时在垃圾回收周期的第1步中回收                                               |
| 诊断选项 | ZConcurrentStringTable          | TRUE   | 允许并发回收字符串表，为true时在垃圾回收周期的第4步中回收，为false时在垃圾回收周期的第3步中回收                                       |
| 诊断选项 | ZConcurrentVMWeakHandles        | TRUE   | 允许并发回收VM弱句柄，为true时在垃圾回收周期的第4步中回收，为false时在垃圾回收周期的第3步中回收                                       |
| 诊断选项 | ZConcurrentJNIWeakGlobalHandles | TRUE   | 允许并发回收JNI弱句柄，为true时在垃圾回收周期的第4步中回收，为false时在垃圾回收周期的第3步中回收                                      |
| 诊断选项 | ZOptimizeLoadBarriers           | TRUE   | 允许C2中对读屏障进行优化，优化的点主要根据数据流优化                                                                                  |
| 开发选项 | ZVerifyLoadBarriers             | FALSE  | 在C2中涉及读屏障时，会对读屏障进行验证                                                                                                |