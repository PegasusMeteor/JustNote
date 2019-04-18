# Goroutine

推荐一本书 [《Concurrency in go 》](https://www.kancloud.cn/mutouzhang/go/596804)

<!-- TOC -->

- [Goroutine](#goroutine)
  - [进程与线程](#%E8%BF%9B%E7%A8%8B%E4%B8%8E%E7%BA%BF%E7%A8%8B)
    - [产生的背景](#%E4%BA%A7%E7%94%9F%E7%9A%84%E8%83%8C%E6%99%AF)
    - [相关概念](#%E7%9B%B8%E5%85%B3%E6%A6%82%E5%BF%B5)
      - [Process](#process)
      - [thread](#thread)
      - [other](#other)
  - [并发（concurrency）和并行（parallelism）](#%E5%B9%B6%E5%8F%91concurrency%E5%92%8C%E5%B9%B6%E8%A1%8Cparallelism)
  - [进程间通信（IPC，Inter-Process Communication）](#%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1ipcinter-process-communication)
  - [线程同步](#%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5)
  - [协程与进程、线程](#%E5%8D%8F%E7%A8%8B%E4%B8%8E%E8%BF%9B%E7%A8%8B%E7%BA%BF%E7%A8%8B)

<!-- /TOC -->

## 进程与线程

在最开始，我们先来老生长谈一下什么是进程、线程。 或许每个人都知道进程和线程，但是有几个人能够快速准确的说出进程是什么呢？  

维基百科中关于 [进程的定义](https://en.wikipedia.org/wiki/Process_(computing))

### 产生的背景

- 由用户输入指令的低效 -->
- 批处理操作系统的串行 -->
- CPU时间片轮转实现操作系统的并发 -->
- 线程实现了进程内部的并发

可以参考 [深入浅出java多线程](https://redspider.gitbook.io/concurrent/di-yi-pian-ji-chu-pian/1)

### 相关概念

#### Process

- 面向进程的操作系统（早期的UNIX，Linux 2.4）是程序执行的基本单位
- 面向线程的操作系统（Linux 2.6之后）进程不是基本单位而是线程的容器
- 进程五种状态（new、running、waiting、ready、terminated）

#### thread

- 操作系统运算调度的最小单位,在进程之中，是进程的实际运行单位.
- 进程中可以有多个执行不同任务的并发线程
- 在Unix System V及SunOS中也被称为轻量进程（lightweight processes），但轻量进程更多指内核线程（kernel thread），而把用户线程（user thread）称为线程。

#### other

- 进程优先级
- 线程优先级

## 并发（concurrency）和并行（parallelism）

“并发”指的是程序的结构，“并行”指的是程序运行时的状态.
并行指物理上同时执行，并发指能够让多个任务在逻辑上交织执行的程序设计.

Concurrency is about dealing with lots of things at once.

Parallelism is about doing lots of things at once.

Not the same, but related.

Concurrency is about structure, parallelism is about execution.

Concurrency provides a way to structure a solution to solve a problem that may (but not necessarily) be parallelizable.

go 官方有一篇bolg [Concurrency is not parallelism](https://blog.golang.org/concurrency-is-not-parallelism)，接下来我们就以官方的slide来认识一下并发和并行。[点击这里](https://talks.golang.org/2012/waza.slide#1)

## 进程间通信（IPC，Inter-Process Communication）

进程是计算机系统分配资源的最小单位（严格说来是线程）。每个进程都有自己的一部分独立的系统资源，彼此是隔离的。为了能使不同的进程互相访问资源并进行协调工作，就有了进程间通信.

主要的IPC方法

- 文件(文件锁 log-->file-->filebeat)
- [信号](https://en.wikipedia.org/wiki/Signal_(IPC)) (SIGILL、SIGKILL、ctrl+c==SIGINT ...)
- [socket](https://en.wikipedia.org/wiki/Berkeley_sockets) (tcp 三次握手)
- [message queue](https://en.wikipedia.org/wiki/Message_queue)
- 管道pipe (ps -aux | grep nginx)
- [命名管道FIFO](https://en.wikipedia.org/wiki/Named_pipe) (mkfifo my_pipe ; gzip -9 -c < my_pipe > out.gz &  ;cat file > my_pipe)  
- [信号量](https://en.wikipedia.org/wiki/Semaphore_(programming)) (信号量与信号不是一个问题，P(wait) 申请,V(signal)释放)
- [共享内存](https://en.wikipedia.org/wiki/Shared_memory)
- [消息传递](https://en.wikipedia.org/wiki/Message_passing)（filebeat --> logstash -->es）
- [内存映射文件](https://en.wikipedia.org/wiki/Memory-mapped_file)

## 线程同步

线程同步被定义为一种机制，可确保两个或多个并发进程或线程不会同时执行称为临界区的某个特定程序段.

- [死锁](https://en.wikipedia.org/wiki/Deadlock)，当许多进程正在等待某个其他进程持有的共享资源（临界区）时发生。在这种情况下，进程只是等待并且不再执行;
- [饥饿](https://en.wikipedia.org/wiki/Starvation_(computer_science)))，当进程等待进入临界区时发生，但其他进程独占关键部分，第一个进程被迫无限期等待;
- [优先级倒置](https://en.wikipedia.org/wiki/Priority_inversion)，在高优先级进程处于临界区时发生，并由中优先级进程中断。这种违反优先权规则的行为可能会在某些情况下发生，并可能导致实时系统的严重后果;
- [忙等待](https://en.wikipedia.org/wiki/Busy_waiting)，当进程经常轮询以确定它是否可以访问关键部分时发生。这种频繁的轮询会拖延其他进程的处理时间。

## 协程与进程、线程
