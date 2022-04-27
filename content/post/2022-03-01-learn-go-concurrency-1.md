---
title: Go并发篇-1-并发原语
date: '2022-03-01'
slug: learn-go-concurrency-1
categories:
  - programing
tags:
  - go
---
go语言的并发编程主要有两种方式：

- 共享内存
  - 并发原语
  - 原子操作
- Channel

使用的策略是：任务编排用 Channel，共享资源保护用传统并发原语。

共享资源。并发地读写共享资源，会出现数据竞争（data race）的问题，所以需要 Mutex、RWMutex 这样的并发原语来保护。

任务编排。需要 goroutine 按照一定的规律执行，而 goroutine 之间有相互等待或者依赖的顺序关系，我们常常使用 WaitGroup 或者 Channel 来实现。

消息传递。信息交流以及不同的 goroutine 之间的线程安全的数据交流，常常使用 Channel 来实现。

---

1. Mutex互斥锁

   - 使用：

   ```go
   type Counter struct {
       mu    sync.Mutex
       Count uint64
   }
   ```

   - 实现：

     1）当Mutex处于正常模式时，若此时没有新goroutine与队头goroutine竞争，则队头goroutine获得。若有新goroutine竞争大概率新goroutine获得。
     2）当队头goroutine竞争锁失败1ms后，它会将Mutex调整为饥饿模式。进入饥饿模式后，锁的所有权会直接从解锁goroutine移交给队头goroutine，此时新来的goroutine直接放入队尾。

     3）当一个goroutine获取锁后，如果发现自己满足下列条件中的任何一个#1它是队列中最后一个#2它等待锁的时间少于1ms，则将锁切换回正常模式

2. RWMutex 

   - 如果你遇到可以明确区分 reader 和 writer goroutine 的场景，且有大量的并发读、少量的并发写，并且有强烈的性能需求，你就可以考虑使用读写锁 RWMutex 替换 Mutex。
   - Go 标准库中的 RWMutex 是基于 Mutex 实现的.
   - 写优先的设计意味着，如果已经有一个 writer 在等待请求锁的话，它会阻止新来的请求锁的 reader 获取到锁，所以优先保障 writer。当然，如果有一些 reader 已经请求了锁的话，新请求的 writer 也会等待已经存在的 reader 都释放锁之后才能获取。所以，写优先级设计中的优先权是针对新来的请求而言的。这种设计主要避免了 writer 的饥饿问题。

3. Cond

   - 较少使用。

4. Once

   - 单例对象的初始化场景。

5. Pool

   - 线程安全。
