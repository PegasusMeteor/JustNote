# 单例模式(Singleton Pattern)

**单例模式(Singleton Pattern)**：确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是一种对象创建型模式。

## 优点

- 在内存里只有一个实例，减少了内存开销
- 可以避免对资源的多重占用
- 设置全局访问点，严格控制访问

## 缺点

- 没有接口，扩展比较困难。

## 重点

- 私有构造器
- 线程安全
- 延迟加载
- 序列化和反序列化安全
- 反射
- DoubleCheck

单例模式有很多种，接下来我们通过演进式的方式来一步一步地实现单例模式的理解。

## 懒汉式单例

### Golang Demo 1

```go
package singleton

type LazySingleton struct {
}

var singleton *LazySingleton

func GetInstense() *LazySingleton {
    if singleton == nil {
        singleton = &LazySingleton{}
    }
    return singleton
}
```

### Java Demo 1

```java
package tech.selinux.design.pattern.creational.singleton;

public class LazySingleton {
  private static LazySingleton lazySingleton = null;

  private LazySingleton() {}

  public static LazySingleton getInstance() {
    if (lazySingleton == null) {
      lazySingleton = new LazySingleton();
    }
    return lazySingleton;
  }
}
```

懒汉式单例模式是最基础的一种写法，但是如果我们在并发环境下去调用GetInstance()的时候，可能会创建出多个instance，这样就违背了我们单例的初衷，所以我们对这个方法进行一个加锁机制，这样就能够保证，每次调用时首先会获取锁资源，保证了实例的单一性。

### Golang Demo 2

```go
package singleton

import "sync"

type LazySingleton struct {
}

var singleton *LazySingleton

var lock sync.Mutex

func GetInstense() *LazySingleton {
    lock.Lock()
    defer lock.Unlock()
    if singleton == nil {
        singleton = &LazySingleton{}
    }
    return singleton
}
```

### Java Demo 2

```java
package tech.selinux.design.pattern.creational.singleton;

public class LazySingleton {
  private static LazySingleton lazySingleton = null;

  private LazySingleton() {}

  public static LazySingleton getInstance() {
    synchronized (LazySingleton.class) {
      if (lazySingleton == null) {
        lazySingleton = new LazySingleton();
      }
      return lazySingleton;
    }
  }

  //  public static synchronized LazySingleton getInstance() {
  //
  //    if (lazySingleton == null) {
  //      lazySingleton = new LazySingleton();
  //    }
  //    return lazySingleton;
  //  }
}
```

代码做了简单的修改了，引入了锁的机制，在GetInstance函数中，每次调用我们都会上一把锁，保证只有一个goroutine执行它，这个时候并发的问题就解决了。不过现在不管什么情况下都会上一把锁，而且加锁的代价是很大的，有没有办法继续对我们的代码进行进一步的优化呢？

## DoubleCheck 懒汉式

### Golang Demo

```go
package singleton

import "sync"

type LazySingleton struct {
}

var singleton *LazySingleton

var lock sync.Mutex

func GetInstense() *LazySingleton {
    if singleton == nil {
        lock.Lock()
        defer lock.Unlock()
        if singleton == nil {
            singleton = &LazySingleton{}
        }
    }

    return singleton
}
```

在上面的方法中，我们做了很多的工作就是为了在并发模式下让，GetInstance中的实例只创建一次。其实在golang中，有一个比较优雅的方式 once.Do 下面我们练下demo。

```golang
package singleton

import "sync"

type LazySingleton struct {
}

var singleton *LazySingleton
var once sync.Once

func GetInstense() *LazySingleton {
    once.Do(func() {
        singleton = &LazySingleton{}
    })
    return singleton
}
```

### Java Demo

```java
package tech.selinux.design.pattern.creational.singleton;

public class LazyDoubleCheckSingleton {
   // volatile 能够解决多线程访问的有序性
  private static volatile LazyDoubleCheckSingleton lazyDoubleCheckSingleton = null;

  private LazyDoubleCheckSingleton() {}

  public static LazyDoubleCheckSingleton getInstance() {
    if (lazyDoubleCheckSingleton == null) {
      synchronized (LazyDoubleCheckSingleton.class) {
        if (lazyDoubleCheckSingleton == null) {
          lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton();
        }
      }
    }
    return lazyDoubleCheckSingleton;
  }
}
```

---

未完，待续...