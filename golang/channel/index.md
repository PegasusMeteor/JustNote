# Channel

<!-- TOC -->

- [Channel](#channel)
  - [Channel定义](#%08channel%E5%AE%9A%E4%B9%89)
  - [make channel](#make-channel)
  - [chansend](#chansend)
  - [chanrecv](#chanrecv)
  - [close](#close)
  - [参考链接](#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

<!-- /TOC -->

## Channel定义

源码位置 [/src/runtime/chan.go](https://github.com/golang/go/blob/master/src/runtime/chan.go)

```go
type hchan struct {
    qcount   uint           // 队列中数据个数
    dataqsiz uint           // channel 大小
    buf      unsafe.Pointer // 存放数据的环形数组
    elemsize uint16         //channel 中数据类型的大小
    closed   uint32         // 表示 channel 是否关闭
    elemtype *_type         // 元素数据类型
    sendx    uint           // send 的数组索引
    recvx    uint           // recv 的数组索引
    recvq    waitq          // 由 recv 行为（也就是 <-ch）阻塞在 channel 上的 goroutine 队列
    sendq    waitq          // 由 send 行为 (也就是 ch<-) 阻塞在 channel 上的 goroutine 队列

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
//
// sudog is necessary because the g ↔ synchronization object relation
// is many-to-many. A g can be on many wait lists, so there may be
// many sudogs for one g; and many gs may be waiting on the same
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
// reflect_makechan 估计是由汇编实现的runtime来调用。
//go:linkname reflect_makechan reflect.makechan
func reflect_makechan(t *chantype, size int) *hchan {
    return makechan(t, size)
}

// makechan64 同上
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
        c.buf = mallocgc(mem, elem, true)
    }

    c.elemsize = uint16(elem.size)
    c.elemtype = elem
    c.dataqsiz = uint(size)

    if debugChan {
        print("makechan: chan=", c, "; elemsize=", elem.size, "; elemalg=", elem.alg, "; dataqsiz=", size, "\n")
    }
    return c
}
```

## chansend

向channel中发送数据。

```go
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
    //
    // After observing that the channel is not closed, we observe that the channel is
    // not ready for sending. Each of these observations is a single word-sized read
    // (first c.closed and second c.recvq.first or c.qcount depending on kind of channel).
    // Because a closed channel cannot transition from 'ready for sending' to
    // 'not ready for sending', even if the channel is closed between the two observations,
    // they imply a moment between the two when the channel was both not yet closed
    // and not ready for sending. We behave as if we observed the channel at that moment,
    // and report that the send cannot proceed.
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

## chanrecv

与发送数据一样，读取数据也要首先获取锁，然后才能读取。

```go
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

    if raceenabled {
        callerpc := getcallerpc()
        racewritepc(c.raceaddr(), callerpc, funcPC(closechan))
        racerelease(c.raceaddr())
    }

    c.closed = 1

    var glist gList

    // release all readers
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


## 参考链接

- [https://www.cnblogs.com/zkweb/p/7815600.html](https://www.cnblogs.com/zkweb/p/7815600.html)
- [http://legendtkl.com/categories/golang](http://legendtkl.com/categories/golang)
- [http://xargin.com/channel-from-usage-to-src-analysis/](http://xargin.com/channel-from-usage-to-src-analysis/)