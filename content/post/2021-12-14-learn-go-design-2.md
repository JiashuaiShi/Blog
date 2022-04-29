---
title: Go设计篇-2-设计队列
date: '2021-12-14'
slug: learn-go-design-2
categories:
  - programing
tags:
  - go
---

## 1 cond是什么？

- Cond 关联的 Locker 实例可以通过 c.L 访问，它内部维护着一个先入先出的等待队列。
- Cond 是为等待 / 通知场景下的并发问题提供支持的。它提供了条件变量的三个基本方法 Signal、Broadcast 和 Wait，为并发的 goroutine 提供等待 / 通知机制。
  - Signal 方法，允许调用者 Caller 唤醒一个等待此 Cond 的 goroutine。如果此时没有等待的 goroutine，显然无需通知 waiter；如果 Cond 等待队列中有一个或者多个等待的 goroutine，则需要从等待队列中移除第一个 goroutine 并把它唤醒。调用 Signal 方法时，不强求持有 c.L 的锁。
  - Broadcast 方法，允许调用者 Caller 唤醒所有等待此 Cond 的 goroutine。如果此时没有等待的 goroutine，显然无需通知 waiter；如果 Cond 等待队列中有一个或者多个等待的 goroutine，则清空所有等待的 goroutine，并全部唤醒。同样地，调用 Broadcast 方法时，也不强求你一定持有 c.L 的锁。
  - Wait 方法，会把调用者 Caller 放入 Cond 的等待队列中并阻塞，直到被 Signal 或者 Broadcast 的方法从等待队列中移除并唤醒。调用 Wait 方法时必须要持有 c.L 的锁。

- WaitGroup 是主 goroutine 等待确定数量的子 goroutine 完成任务；而 Cond 是等待某个条件满足，这个条件的修改可以被任意多的 goroutine 更新，而且 Cond 的 Wait 不关心也不知道其他 goroutine 的数量，只关心等待条件。而且 Cond 还有单个通知的机制，也就是 Signal 方法。

## 2 如何使用cond设计简单队列？

- 生产者使用cond加锁获取队列，然后往队列中添加变量。然后广播通知消费者。进入间隔休眠。
- 消费者加锁拿到队列，如果为空，则持续阻塞等待，进入到cond的等待队列。此时生产者重新获得锁，添加变量。
- 当队列不为空时，根据FIFO，消费者从队头取出变量。进入间隔休眠。

```go
type Queue struct {
	queue []string
	cond  *sync.Cond
}

func main() {
	q := Queue{
		queue: []string{},
		cond:  sync.NewCond(&sync.Mutex{}),
	}
	go func() {
		for {
			q.Enqueue("a")
			time.Sleep(time.Second * 2)
		}
	}()
	for {
		q.Dequeue()
		time.Sleep(time.Second)
	}
}

func (q *Queue) Enqueue(item string) {
	q.cond.L.Lock()
	defer q.cond.L.Unlock()
	q.queue = append(q.queue, item)
	fmt.Printf("putting %s to queue, notify all\n", item)
	q.cond.Broadcast()
}

func (q *Queue) Dequeue() string {
	q.cond.L.Lock()
	defer q.cond.L.Unlock()
	for len(q.queue) == 0 {
		fmt.Println("no data available, wait")
		q.cond.Wait()
	}
	result := q.queue[0]
	q.queue = q.queue[1:]
	return result
}

```

