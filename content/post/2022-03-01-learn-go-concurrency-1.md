---
title: Go并发篇-1-并发原语
date: '2022-03-01'
slug: learn-go-concurrency-1
categories:
  - programing
tags:
  - go
---
## 并发编程的方式

go语言的并发编程主要有两种方式：

- 共享内存（并发原语、原子操作）
- Channel

使用的策略是：共享资源保护用传统并发原语，任务编排、消息传递用 Channel，

- 共享资源：并发地读写共享资源，会出现数据竞争（data race）的问题，所以需要 Mutex、RWMutex 这样的并发原语来保护。

- 任务编排：需要 goroutine 按照一定的规律执行，而 goroutine 之间有相互等待或者依赖的顺序关系，我们常常使用 WaitGroup 或者 Channel 来实现。

- 消息传递：信息交流以及不同的 goroutine 之间的线程安全的数据交流，常常使用 Channel 来实现。

## 基本原语的使用

### Mutex

1. **使用方法：**

   一般把需要保护的资源对象和mutex封装到一个结构体。读写资源时先加锁。

   ```go
   type Counter struct {
       mu    sync.Mutex
       Count uint64
   }
   ```

2. **实现策略：**

   1. 当Mutex处于正常模式时，若此时没有新goroutine与队头goroutine竞争，则队头goroutine获得。若有新goroutine竞争大概率新goroutine获得。
   2. 当队头goroutine竞争锁失败1ms后，它会将Mutex调整为饥饿模式。进入饥饿模式后，锁的所有权会直接从解锁goroutine移交给队头goroutine，此时新来的goroutine直接放入队尾。
   3. 当一个goroutine获取锁后，如果发现自己满足下列条件中的任何一个#1它是队列中最后一个#2它等待锁的时间少于1ms，则将锁切换回正常模式

3. **使用注意：**
   - mutex是不能复制的。关键是在并发环境下，你根本不知道要复制的 Mutex 状态是什么，因为要复制的 Mutex 是由其它 goroutine 并发访问的，状态可能总是在变化。
   - mutex不是可重入的锁。拥有锁的线程，可再次申请锁。不会阻塞，可直接返回。被称为可重入锁或者递归锁。
   - 检查工具：
     - 运行时检查。 checkdead()。
     - 编译期检查。vet工具。检查是通过copylock分析器静态分析实现的。这个分析器会分析函数调用、range 遍历、复制、声明、函数返回值等位置，有没有锁的值 copy 的情景，以此来判断有没有问题。只要是实现了 Locker 接口，就会被分析。

### RWMutex 

1. **使用场景：**

   如果你遇到可以明确区分 reader 和 writer goroutine 的场景，且有大量的并发读、少量的并发写，并且有强烈的性能需求，你就可以考虑使用读写锁 RWMutex 替换 Mutex。

2. **实现策略：**

   - Go 标准库中的 RWMutex 是基于 Mutex 实现的.

   - 写优先的设计主要避免了 writer 的饥饿问题。如果已经有一个 writer 在等待请求锁的话，它会阻止新来的请求锁的 reader 获取到锁，所以优先保障 writer。当然，如果有一些 reader 已经请求了锁的话，新请求的 writer 也会等待已经存在的 reader 都释放锁之后才能获取。所以，写优先级设计中的优先权是针对新来的请求而言的。


### Cond

- 生产者-消费者队列模型。

### Once
1. **实现原理：**

   使用互斥锁。双重检查。

3. **使用场景：**单例对象的初始化场景。

```go
type Singleton struct {
	data string
}

var singleInstance *Singleton
var once sync.Once

func GetSingletonObj() *Singleton {
	once.Do(func() {
		fmt.Println("Create Obj")
		singleInstance = new(Singleton)
	})
	return singleInstance
}
```

### Pool

1. **使用场景：**

   - 频繁创建的对象，创建成本较高的场景下使用。

   - 线程安全，可以并发调用，池化的对象可能会被垃圾回收掉，这对于数据库长连接等场景是不合适的。

   - sync.Pool 不可在使用之后再复制使用。


2. **使用示例：**

   - Go 内部库也用到了 sync.Pool，比如 fmt 包，它会使用一个动态大小的 buffer 池做输出缓存，当大量的 goroutine 并发输出的时候，就会创建比较多的 buffer，并且在不需要的时候回收掉。

   - 标准库的http client本身也是使用了池化的设计。


3. **常用的第三方连接池：**

   - worker pool，gammazero/workerpool 、grpool 、dpaks/goworkers


   - fatih/pool tcp的连接池。 它通过把 net.Conn 包装成 PoolConn，实现了拦截 net.Conn 的 Close 方法，避免了真正地关闭底层连接，而是把这个连接放回到池中。它的 Pool 是通过 Channel 实现的，空闲的连接放入到 Channel 中。

   - 数据库连接池：标准库 sql.DB 还提供了一个通用的数据库的连接池。DB 的 freeConn 保存了 idle 的连接，这样，当我们获取数据库连接的时候，它就会优先尝试从 freeConn 获取已有的连接。


### WaitGroup

-   使用 WaitGroup 的正确姿势是，预先确定好 WaitGroup 的计数值，然后调用相同次数的 Done 完成相应的任务，等所有的 Add 方法调用之后再调用 Wait。
-   计数器不能设置为负值。
-   WaitGroup 是可以重用的。前一个 Wait 还没结束就重用 WaitGroup会出现错误。