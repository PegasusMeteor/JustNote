# Channel

## Channel定义

源码位置 /src/runtime/chan.go

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

sudog 源码位置 src/runtime/runtime2.go  
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

想channel中发送数据。

```go

```

## 参考链接

- [https://www.cnblogs.com/zkweb/p/7815600.html](https://www.cnblogs.com/zkweb/p/7815600.html)
- [http://legendtkl.com/categories/golang](http://legendtkl.com/categories/golang)