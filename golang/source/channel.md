# Channel

<!-- TOC -->

- [Channel](#channel)
  - [提出问题](#%E6%8F%90%E5%87%BA%E9%97%AE%E9%A2%98)
  - [Channel定义](#%08channel%E5%AE%9A%E4%B9%89)
  - [make channel](#make-channel)
  - [chansend](#chansend)
  - [chanrecv](#chanrecv)
  - [close](#close)
  - [参考链接](#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

<!-- /TOC -->

## 提出问题

经过自己很长一段时间的实验，发现带着问题来阅读源码是非常高效的，因为源码一般大而杂，分支也比较多，稍不留神就容易走进细枝末节跳不出来了。所以我们这里先抛出几个问题，然后带着问题一起去源码中寻找答案。  

- channel的数据结构是怎么，内部又包含了哪些组件
- goroutine向channel中发送数据的阻塞机制是什么
- 有缓冲和无缓冲在实现过程中有哪些区别
- channel-safe的实现机制是什么
- write-to-channel 与 read-from-channel 是怎么协作的
- channel在关闭之后还可以做到能读不能写，那关闭时需要注意什么

接下来我们将一些比较重要的源码，以及源码分析过程中的备注都贴到下方。

## Channel定义

源码位置 [/src/runtime/chan.go](https://github.com/golang/go/blob/master/src/runtime/chan.go)

```go
type hchan struct {
    qcount   uint           // qcount 是 buf 中已塞进的元素数量,当前队列中的元素数量
    dataqsiz uint           // 队列可以容纳的元素数量, 如果为0表示这个channel无缓冲区，就是make(chan,size)中的size
    buf      unsafe.Pointer // 队列的缓冲区, 结构是环形队列
    elemsize uint16         //channel 中数据类型的大小
    closed   uint32         // 表示 channel 是否关闭
    elemtype *_type         // 元素的类型, 判断是否调用写屏障时使用
    sendx    uint           // 发送元素的序号
    recvx    uint           // 接收元素的序号
    recvq    waitq          // 当前等待从channel接收数据的G的链表(实际类型是sudog的链表),由 recv 行为（也就是 <-ch）阻塞在 channel 上的 goroutine 队列
    sendq    waitq          // 当前等待发送数据到channel的G的链表(实际类型是sudog的链表),由 send 行为 (也就是 ch<-) 阻塞在 channel 上的 goroutine 队列

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex
}

type waitq struct {
    first *sudog
    last  *sudog
}
```

channel 的实现，很明显是使用队列来完成。

大家都知道channel是线程安全的，那么它是怎么实现的呢？答案很简单就是锁机制。

sudog 源码位置 [src/runtime/runtime2.go](https://github.com/golang/go/blob/master/src/runtime/runtime2.go)  
sudog 是一种非常重要的数据结构，相当于我们前面介绍的G。

```go
// sudog represents a g in a wait list, such as for sending/receiving
// on a channel.
// sudog 表示一个在 wait list 中的g
// sudog is necessary because the g ↔ synchronization object relation
// is many-to-many.
// A g can be on many wait lists, so there may be
// many sudogs for one g;
//
// and many gs may be waiting on the same
// synchronization object, so there may be many sudogs for one object.
//
// sudogs are allocated from a special pool. Use acquireSudog and
// releaseSudog to allocate and free them.
type sudog struct {
    // The following fields are protected by the hchan.lock of the
    // channel this sudog is blocking on. shrinkstack depends on
    // this for sudogs involved in channel ops.

    g *g

    // isSelect indicates g is participating in a select, so
    // g.selectDone must be CAS'd to win the wake-up race.
    isSelect bool
    next     *sudog
    prev     *sudog
    elem     unsafe.Pointer // data element (may point to stack)

    // The following fields are never accessed concurrently.
    // For channels, waitlink is only accessed by g.
    // For semaphores, all fields (including the ones above)
    // are only accessed when holding a semaRoot lock.

    acquiretime int64
    releasetime int64
    ticket      uint32
    parent      *sudog // semaRoot binary tree
    waitlink    *sudog // g.waiting list or semaRoot
    waittail    *sudog // semaRoot
    c           *hchan // channel
}
```

## make channel

创建一个go channel

```go
//go:linkname reflect_makechan reflect.makechan
func reflect_makechan(t *chantype, size int) *hchan {
    return makechan(t, size)
}

func makechan64(t *chantype, size int64) *hchan {
    if int64(int(size)) != size {
        panic(plainError("makechan: size out of range"))
    }

    return makechan(t, int(size))
}

// 创建一个go channel
func makechan(t *chantype, size int) *hchan {
    // 传入的数据类型
    elem := t.elem

    // compiler checks this but be safe.
    if elem.size >= 1<<16 {
        throw("makechan: invalid channel element type")
    }
    if hchanSize%maxAlign != 0 || elem.align > maxAlign {
        throw("makechan: bad alignment")
    }
    // 根据传入的数据类型占用的地址空间大小和缓存个数计算一个内存地址
    mem, overflow := math.MulUintptr(elem.size, uintptr(size))
    if overflow || mem > maxAlloc-hchanSize || size < 0 {
        panic(plainError("makechan: size out of range"))
    }

    // Hchan does not contain pointers interesting for GC when elements stored in buf do not contain pointers.
    // buf points into the same allocation, elemtype is persistent.
    // SudoG's are referenced from their owning thread so they can't be collected.
    // TODO(dvyukov,rlh): Rethink when collector can move allocated objects.
    var c *hchan
    switch {
    case mem == 0:
        // Queue or element size is zero.
        c = (*hchan)(mallocgc(hchanSize, nil, true))
        // Race detector uses this location for synchronization.
        c.buf = c.raceaddr()
    case elem.ptrdata == 0:
        // Elements do not contain pointers.
        // Allocate hchan and buf in one call.
        c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
        c.buf = add(unsafe.Pointer(c), hchanSize)
    default:
        // Elements contain pointers.
        c = new(hchan)
        // 根据channel中存储的数据类型，分配的一段内存空间
        c.buf = mallocgc(mem, elem, true)
    }

    c.elemsize = uint16(elem.size)
    c.elemtype = elem
    // channel 缓存大小
    c.dataqsiz = uint(size)

    if debugChan {
        print("makechan: chan=", c, "; elemsize=", elem.size, "; elemalg=", elem.alg, "; dataqsiz=", size, "\n")
    }
    return c
}

```

makechan 这里有几个需要注意的点。  

- 根据channel中传递的元素类型(`makechan(type,size)`)，计算出一块内存大小，后期创建一块内存区域, 对应代码是

```go
mem, overflow := math.MulUintptr(elem.size, uintptr(size))
```

- `qcount` 记录的是buf 中的数据数量，后面会有增减。`dataqsiz` 记录的是 `makechan(type,size)` 中的size，不会变。要记住，要不然后面会混。

## chansend

向channel中发送数据。

```go
// channel send 数据的入口
// entry point for c <- x from compiled code
//go:nosplit
func chansend1(c *hchan, elem unsafe.Pointer) {
    chansend(c, elem, true, getcallerpc())
}

/*
 * generic single channel send/recv
 * If block is not nil,
 * then the protocol will not
 * sleep but return if it could
 * not complete.
 *
 * sleep can wake up with g.param == nil
 * when a channel involved in the sleep has
 * been closed.  it is easiest to loop and re-run
 * the operation; we'll see that it's now closed.
 */
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // 应用层的 channel 为空
    // 例如 var a chan int
    // a<-1
    if c == nil {
        if !block {
            return false
        }
        // nil channel 发送数据会永远阻塞下去
        // PS：注意，会发生 panic 那种情况是 channel 被 closed 了，不是 nil channel
        // 挂起当前 goroutine
        gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
        throw("unreachable")
    }

    // 开发调试时使用，不用管
    if debugChan {
        print("chansend: chan=", c, "\n")
    }
    // 调试的，可忽略
    if raceenabled {
        racereadpc(c.raceaddr(), callerpc, funcPC(chansend))
    }

    // Fast path: check for failed non-blocking operation without acquiring the lock.
    // 检查失败的非阻塞操作而不获取锁定。
    //
    // After observing that the channel is not closed, we observe that the channel is
    // not ready for sending.
    // 在观察到通道未关闭后，我们发现通道尚未准备好发送。
    // Each of these observations is a single word-sized read
    // (first c.closed and second c.recvq.first or c.qcount depending on kind of channel).

    // 这些观察中的每一个都是单个字大小的读取（第一个c.closed和第二个c.recvq.first或c.qcount，取决于通道的类型）。
    // Because a closed channel cannot transition(转换) from 'ready for sending' to
    // 'not ready for sending', even if the channel is closed between the two observations,
    // they imply(意味着) a moment between the two when the channel was both not yet closed
    // and not ready for sending. We behave as if we observed the channel at that moment,
    // and report that the send cannot proceed.
    //
    //
    // It is okay if the reads are reordered here: if we observe that the channel is not
    // ready for sending and then observe that it is not closed, that implies that the
    // channel wasn't closed during the first observation.

    if !block && c.closed == 0 && ((c.dataqsiz == 0 && c.recvq.first == nil) ||
        (c.dataqsiz > 0 && c.qcount == c.dataqsiz)) {
        return false
    }

    var t0 int64
    if blockprofilerate > 0 {
        t0 = cputicks()
    }

    // 加锁了所以并发安全
    lock(&c.lock)

    // channel 已被关闭，panic
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("send on closed channel"))
    }

    // 这是一个出队列的操作
    // 一单这里进去了，说明buf已空
    if sg := c.recvq.dequeue(); sg != nil {
        // 寻找一个等待中的 receiver
        // 越过 channel 的 buffer
        // 直接把要发的数据拷贝给这个 receiver
        // 然后就返
        // 将锁也一并传递给了send
        // Found a waiting receiver. We pass the value we want to send
        // directly to the receiver, bypassing the channel buffer (if any).
        send(c, sg, ep, func() { unlock(&c.lock) }, 3)
        return true
    }

    // qcount 是 buffer 中已塞进的元素数量
    // dataqsize 是 buffer 的总大小
    // 说明还有余量
    if c.qcount < c.dataqsiz {
        // 存入内存buf
        // Space is available in the channel buffer. Enqueue the element to send.
        qp := chanbuf(c, c.sendx)
        if raceenabled {
            raceacquire(qp)
            racerelease(qp)
        }
        // 将 goroutine 的数据拷贝到 buffer 中
        typedmemmove(c.elemtype, qp, ep)
        c.sendx++
        // 环形队列，所以如果已经加到最大了，就回 0
        if c.sendx == c.dataqsiz {
            c.sendx = 0
        }
        // 将 buffer 的元素计数 +1
        c.qcount++
        unlock(&c.lock)
        return true
    }

    if !block {
        unlock(&c.lock)
        return false
    }

    // Block on the channel. Some receiver will complete our operation for us.
    // 在 channel 上阻塞，receiver 会帮我们完成后续的工作
    gp := getg()
    mysg := acquireSudog()
    mysg.releasetime = 0
    if t0 != 0 {
        mysg.releasetime = -1
    }
    // No stack splits between assigning elem and enqueuing mysg
    // on gp.waiting where copystack can find it.
    mysg.elem = ep
    mysg.waitlink = nil
    mysg.g = gp
    mysg.isSelect = false
    mysg.c = c
    gp.waiting = mysg
    gp.param = nil
    // 将当前这个发送 goroutine 打包后的 sudog 入队到 channel 的 sendq 队列中
    // 进入队列
    c.sendq.enqueue(mysg)

    // 将这个发送 g 从 Grunning -> Gwaiting
    // 进入休眠
    goparkunlock(&c.lock, waitReasonChanSend, traceEvGoBlockSend, 3)
    // Ensure the value being sent is kept alive until the
    // receiver copies it out. The sudog has a pointer to the
    // stack object, but sudogs aren't considered as roots of the
    // stack tracer.
    KeepAlive(ep)

    // someone woke us up.
    // 这里是被唤醒后要执行的代码
    if mysg != gp.waiting {
        // 先判断当前是不是合法的休眠中
        throw("G waiting list is corrupted")
    }
    gp.waiting = nil
    if gp.param == nil {
        if c.closed == 0 {
            throw("chansend: spurious wakeup")
        }
        // 唤醒后发现 channel 被人关了，panic
        panic(plainError("send on closed channel"))
    }
    gp.param = nil
    if mysg.releasetime > 0 {
        blockevent(mysg.releasetime-t0, 2)
    }
    mysg.c = nil
    releaseSudog(mysg)
    return true
}

// send processes a send operation on an empty channel c.
// The value ep sent by the sender is copied to the receiver sg.
// The receiver is then woken up to go on its merry way.
// Channel c must be empty and locked.  send unlocks c with unlockf.
// sg must already be dequeued from c.
// ep must be non-nil and point to the heap or the caller's stack.
func send(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func(), skip int) {
    //忽略
    if raceenabled {
        if c.dataqsiz == 0 {
            racesync(c, sg)
        } else {
            // Pretend we go through the buffer, even though
            // we copy directly. Note that we need to increment
            // the head/tail locations only when raceenabled.
            qp := chanbuf(c, c.recvx)
            raceacquire(qp)
            racerelease(qp)
            raceacquireg(sg.g, qp)
            racereleaseg(sg.g, qp)
            c.recvx++
            if c.recvx == c.dataqsiz {
                c.recvx = 0
            }
            c.sendx = c.recvx // c.sendx = (c.sendx+1) % c.dataqsiz
        }
    }

    // receiver 的 sudog 已经在对应区域分配过空间
    // 我们只要把数据拷贝过去
    if sg.elem != nil {
        sendDirect(c.elemtype, sg, ep)
        sg.elem = nil
    }
    gp := sg.g
    unlockf()
    gp.param = unsafe.Pointer(sg)
    if sg.releasetime != 0 {
        sg.releasetime = cputicks()
    }
    // Gwaiting -> Grunnable
    goready(gp, skip+1)
}
```

前面我们提到过，channel线程安全的原因就是有锁机制，所以向channel中写入数据的话，首先要获取锁，然后发送数据，发送完成之后，释放锁。  

通过代码我们能看出，发送数据时，首先会查看recvq中是否有是否有goroutine在等待，如果有直接讲数据发送给他。

发送数据到channel实际调用的是runtime.chansend1函数, chansend1函数调用了chansend函数, 流程是:

> 下面的文字引用自 [协程的实现原理](https://www.cnblogs.com/zkweb/p/7815600.html)

- 检查channel.recvq是否有等待中的接收者的G
  - 如果有, 表示channel无缓冲区或者缓冲区为空
  - 调用send函数
    - 如果sudog.elem不等于nil, 调用sendDirect函数从发送者直接复制元素
    - 等待接收的sudog.elem是指向接收目标的内存的指针, 如果是接收目标是_则elem是nil, 可以省略复制
    - 等待发送的sudog.elem是指向来源目标的内存的指针
    - 复制后调用goready恢复发送者的G
      - 切换到g0调用ready函数, 调用完切换回来
        - 把G的状态由等待中(_Gwaiting)改为待运行(_Grunnable)
        - 把G放到P的本地运行队列
        - 如果当前有空闲的P, 但是无自旋的M(nmspinning等于0), 则唤醒或新建一个M
  - 从发送者拿到数据并唤醒了G后, 就可以从chansend返回了
- 判断是否可以把元素放到缓冲区中
  - 如果缓冲区有空余的空间, 则把元素放到缓冲区并从chansend返回
- 无缓冲区或缓冲区已经写满, 发送者的G需要等待
  - 获取当前的g
  - 新建一个sudog
  - 设置sudog.elem = 指向发送内存的指针
  - 设置sudog.g = g
  - 设置sudog.c = channel
  - 设置g.waiting = sudog
  - 把sudog放入channel.sendq
  - 调用goparkunlock函数
    - 调用gopark函数
      - 通过mcall函数调用park_m函数
        - mcall函数和上面说明的一样, 会把当前的状态保存到g.sched, 然后切换到g0和g0的栈空间并执行指定的函数
        - park_m函数首先把G的状态从运行中(_Grunning)改为等待中(_Gwaiting)
        - 然后调用dropg函数解除M和G之间的关联
        - 再调用传入的解锁函数, 这里的解锁函数会对解除channel.lock的锁定
        - 最后调用schedule函数继续调度
- 从这里恢复表示已经成功发送或者channel已关闭
  - 检查sudog.param是否为nil, 如果为nil表示channel已关闭, 抛出panic
  - 否则释放sudog然后返回

## chanrecv

与发送数据一样，读取数据也要首先获取锁，然后才能读取。

```go

// entry points for <- c from compiled code
//go:nosplit
func chanrecv1(c *hchan, elem unsafe.Pointer) {
    chanrecv(c, elem, true)
}

//go:nosplit
func chanrecv2(c *hchan, elem unsafe.Pointer) (received bool) {
    _, received = chanrecv(c, elem, true)
    return
}

// chanrecv receives on channel c and writes the received data to ep.
// ep may be nil, in which case received data is ignored.
// If block == false and no elements are available, returns (false, false).
// Otherwise, if c is closed, zeros *ep and returns (true, false).
// Otherwise, fills in *ep with an element and returns (true, true).
// A non-nil ep must point to the heap or the caller's stack.
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
    // raceenabled: don't need to check ep, as it is always on the stack
    // or is new memory allocated by reflect.

    if debugChan {
        print("chanrecv: chan=", c, "\n")
    }
    // 如果在 nil channel 上进行 recv 操作，那么会永远阻塞
    if c == nil {
        if !block {
            // 非阻塞的情况下
            // 要直接返回
            // 非阻塞出现在一些 select 的场景中
            // 参见 selectnbrecv/selectnbrecv2
            return
        }
        // 当前 goroutine: Grunning -> Gwaiting
        // 其实就是该 goroutine 直接泄露 leak 了
        gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
        throw("unreachable")
    }

    // Fast path: check for failed non-blocking operation without acquiring the lock.
    //
    // After observing that the channel is not ready for receiving, we observe that the
    // channel is not closed. Each of these observations is a single word-sized read
    // (first c.sendq.first or c.qcount, and second c.closed).
    // Because a channel cannot be reopened, the later observation of the channel
    // being not closed implies that it was also not closed at the moment of the
    // first observation. We behave as if we observed the channel at that moment
    // and report that the receive cannot proceed.
    //
    // The order of operations is important here: reversing the operations can lead to
    // incorrect behavior when racing with a close.
    if !block && (c.dataqsiz == 0 && c.sendq.first == nil ||
        c.dataqsiz > 0 && atomic.Loaduint(&c.qcount) == 0) &&
        atomic.Load(&c.closed) == 0 {
        // 非阻塞且没内容可收的情况下要直接返回
        // 两个 bool 的零值就是 false，false
        return
    }

    var t0 int64
    if blockprofilerate > 0 {
        t0 = cputicks()
    }

    lock(&c.lock)

    // 当前 channel 中没有数据可读
    // 直接返回 not selected
    if c.closed != 0 && c.qcount == 0 {
        if raceenabled {
            raceacquire(c.raceaddr())
        }
        unlock(&c.lock)
        if ep != nil {
            typedmemclr(c.elemtype, ep)
        }
        return true, false
    }

    // sender 队列中有 sudog 在等待
    // 直接从该 sudog 中获取数据拷贝到当前 g 即可
    // 一旦执行到这里，说明buf已满
    if sg := c.sendq.dequeue(); sg != nil {
        // Found a waiting sender. If buffer is size 0, receive value
        // directly from sender. Otherwise, receive from head of queue
        // and add sender's value to the tail of the queue (both map to
        // the same buffer slot because the queue is full).
        recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
        return true, true
    }

    if c.qcount > 0 {
        // Receive directly from queue
        qp := chanbuf(c, c.recvx)
        if raceenabled {
            raceacquire(qp)
            racerelease(qp)
        }
        // 直接从 buffer 里拷贝数据
        if ep != nil {
            typedmemmove(c.elemtype, ep, qp)
        }
        typedmemclr(c.elemtype, qp)
        c.recvx++
        if c.recvx == c.dataqsiz {
            c.recvx = 0
        }
        c.qcount--
        unlock(&c.lock)
        return true, true
    }

    if !block {
        unlock(&c.lock)
        // 非阻塞时，且无数据可收
        // 始终不选中，这是在 buffer 中没内容的时候
        return false, false
    }

    // no sender available: block on this channel.
    gp := getg()
    mysg := acquireSudog()
    mysg.releasetime = 0
    if t0 != 0 {
        mysg.releasetime = -1
    }
    // No stack splits between assigning elem and enqueuing mysg
    // on gp.waiting where copystack can find it.
    mysg.elem = ep
    mysg.waitlink = nil
    gp.waiting = mysg
    mysg.g = gp
    mysg.isSelect = false
    mysg.c = c
    gp.param = nil
    c.recvq.enqueue(mysg)
    goparkunlock(&c.lock, waitReasonChanReceive, traceEvGoBlockRecv, 3)

    // someone woke us up
    if mysg != gp.waiting {
        throw("G waiting list is corrupted")
    }
    gp.waiting = nil
    if mysg.releasetime > 0 {
        blockevent(mysg.releasetime-t0, 2)
    }
    closed := gp.param == nil
    gp.param = nil
    mysg.c = nil
    releaseSudog(mysg)
    // 如果 channel 未被关闭，那就是真的 recv 到数据了
    return true, !closed
}

// recv processes a receive operation on a full channel c.
// There are 2 parts:
// 1) The value sent by the sender sg is put into the channel
//    and the sender is woken up to go on its merry way.
// 2) The value received by the receiver (the current G) is
//    written to ep.
// For synchronous channels, both values are the same.
// For asynchronous channels, the receiver gets its data from
// the channel buffer and the sender's data is put in the
// channel buffer.
// Channel c must be full and locked. recv unlocks c with unlockf.
// sg must already be dequeued from c.
// A non-nil ep must point to the heap or the caller's stack.
func recv(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func(), skip int) {
    if c.dataqsiz == 0 {
        if raceenabled {
            racesync(c, sg)
        }
        if ep != nil {
            // copy data from sender
            recvDirect(c.elemtype, sg, ep)
        }
    } else {
        // Queue is full. Take the item at the
        // head of the queue. Make the sender enqueue
        // its item at the tail of the queue. Since the
        // queue is full, those are both the same slot.
        qp := chanbuf(c, c.recvx)
        if raceenabled {
            raceacquire(qp)
            racerelease(qp)
            raceacquireg(sg.g, qp)
            racereleaseg(sg.g, qp)
        }
        // copy data from queue to receiver
        if ep != nil {
            typedmemmove(c.elemtype, ep, qp)
        }
        // copy data from sender to queue
        typedmemmove(c.elemtype, qp, sg.elem)
        c.recvx++
        if c.recvx == c.dataqsiz {
            c.recvx = 0
        }
        c.sendx = c.recvx // c.sendx = (c.sendx+1) % c.dataqsiz
    }
    sg.elem = nil
    gp := sg.g
    unlockf()
    gp.param = unsafe.Pointer(sg)
    if sg.releasetime != 0 {
        sg.releasetime = cputicks()
    }
    goready(gp, skip+1)
}

```

**画重点:**

下面这段代码很重要，它解释了sendx和recvx的大小，最大recvx不会超过dataqsiz

```go
if c.recvx == c.dataqsiz {
    c.recvx = 0
}
c.sendx = c.recvx // c.sendx = (c.sendx+1) % c.dataqsiz
```

> 下面的文字引用自 [协程的实现原理](https://www.cnblogs.com/zkweb/p/7815600.html)

从channel接收数据实际调用的是runtime.chanrecv1函数, chanrecv1函数调用了chanrecv函数, 流程是:

- 检查channel.sendq中是否有等待中的发送者的G
  - 如果有, 表示channel无缓冲区或者缓冲区已满, 这两种情况需要分别处理(为了保证入出队顺序一致)
  - 调用recv函数
    - 如果无缓冲区, 调用recvDirect函数把元素直接复制给接收者
    - 如果有缓冲区代表缓冲区已满
      - 把队列中下一个要出队的元素直接复制给接收者
      - 把发送的元素复制到队列中刚才出队的位置
      - 这时候缓冲区仍然是满的, 但是发送序号和接收序号都会增加1
    - 复制后调用goready恢复接收者的G, 处理同上
  - 把数据交给接收者并唤醒了G后, 就可以从chanrecv返回了
- 判断是否可以从缓冲区获取元素
  - 如果缓冲区有元素, 则直接取出该元素并从chanrecv返回
- 无缓冲区或缓冲区无元素, 接收者的G需要等待
  - 获取当前的g
  - 新建一个sudog
  - 设置sudog.elem = 指向接收内存的指针
  - 设置sudog.g = g
  - 设置sudog.c = channel
  - 设置g.waiting = sudog
  - 把sudog放入channel.recvq
  - 调用goparkunlock函数, 处理同上
- 从这里恢复表示已经成功接收或者channel已关闭
  - 检查sudog.param是否为nil, 如果为nil表示channel已关闭
  - 和发送不一样的是接收不会抛panic, 会通过返回值通知channel已关闭
  - 释放sudog然后返回

## close

```go


func closechan(c *hchan) {
    // 关闭一个 nil channel 会直接 panic
    if c == nil {
        panic(plainError("close of nil channel"))
    }
    // 上锁，这个锁的粒度比较大，一直到释放完所有的 sudog 才解锁
    lock(&c.lock)
    // 在 close channel 时，如果 channel 已经关闭过了
    // 直接触发 panic
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("close of closed channel"))
    }

    // 忽略
    if raceenabled {
        callerpc := getcallerpc()
        racewritepc(c.raceaddr(), callerpc, funcPC(closechan))
        racerelease(c.raceaddr())
    }

    // 先讲 close 标志置为1
    c.closed = 1

    var glist gList

    // release all readers
    // 将所有的recvq 出队列
    for {
        sg := c.recvq.dequeue()
        // 弹出的 sudog 是 nil
        // 说明读队列已经空了
        if sg == nil {
            break
        }
        // sg.elem unsafe.Pointer，指向 sudog 的数据元素
        // 该元素可能在堆上分配，也可能在栈上
        if sg.elem != nil {
            // 释放对应的内存
            typedmemclr(c.elemtype, sg.elem)
            sg.elem = nil
        }
        if sg.releasetime != 0 {
            sg.releasetime = cputicks()
        }
        // 将 goroutine 入 glist
        // 为最后将全部 goroutine 都 ready 做准备
        gp := sg.g
        gp.param = nil
        if raceenabled {
            raceacquireg(gp, c.raceaddr())
        }
        glist.push(gp)
    }

    // release all writers (they will panic)
    // 将所有挂在 channel 上的 writer 从 sendq 中弹出
    // 该操作会使所有 writer panic
    for {
        sg := c.sendq.dequeue()
        if sg == nil {
            break
        }
        sg.elem = nil
        if sg.releasetime != 0 {
            sg.releasetime = cputicks()
        }
        // 将 goroutine 入 glist
        // 为最后将全部 goroutine 都 ready 做准备
        gp := sg.g
        gp.param = nil
        if raceenabled {
            raceacquireg(gp, c.raceaddr())
        }
        glist.push(gp)
    }
    // 在释放所有挂在 channel 上的读或写 sudog 时
    // 是一直在临界区的
    unlock(&c.lock)

    // Ready all Gs now that we've dropped the channel lock.
    for !glist.empty() {
        gp := glist.pop()
        gp.schedlink = 0
        goready(gp, 3)
    }
}

```

> 下面的文字引用自 [协程的实现原理](https://www.cnblogs.com/zkweb/p/7815600.html)

关闭channel实际调用的是closechan函数, 流程是:

- 设置channel.closed = 1
- 枚举channel.recvq, 清零它们sudog.elem, 设置sudog.param = nil
- 枚举channel.sendq, 设置sudog.elem = nil, 设置sudog.param = nil
- 调用goready函数恢复所有接收者和发送者的G

## 参考链接

- [协程的实现原理](https://www.cnblogs.com/zkweb/p/7815600.html)
- [Go Channel 源码剖析](http://legendtkl.com/categories/golang)
- [Channel 从使用到源码分析](https://github.com/cch123/golang-notes/blob/master/channel.md)
- [理解go channel](https://blog.lab99.org/post/golang-2017-10-04-video-understanding-channels.html#select)
- [Go Chanel 使用与原理 三](https://segmentfault.com/a/1190000018531960)