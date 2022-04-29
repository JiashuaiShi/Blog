---
title: Go设计篇-4-设计实现最小化ConcurrentMap
date: '2021-12-16'
slug: learn-go-design-4
categories:
  - programing
tags:
  - go
---
## 设计考虑：
- key 类型的 K 必须是可比较的（comparable），也就是可以通过 == 和 != 操作符进行比较；value 的值和类型无所谓，可以是任意的类型，或者为 nil。
- map的类型支持基本类型，通过interface{}或者泛型来设计。
- map支持的方法：
  - 读操作：遍历、查询。(加读锁)
  - 写操作：增加、修改、删除。(加写锁)
- 并发Map类型的两种方案：
  - 对整个Map加锁。
  - 对Map分片加锁。

## 全局锁实现：

```go
type RWMap struct { // 一个读写锁保护的线程安全的map
    sync.RWMutex // 读写锁保护下面的map字段
    m map[int]int
}
// 新建一个RWMap
func NewRWMap(n int) *RWMap {
    return &RWMap{
        m: make(map[int]int, n),
    }
}
func (m *RWMap) Get(k int) (int, bool) { //从map中读取一个值
    m.RLock()
    defer m.RUnlock()
    v, existed := m.m[k] // 在锁的保护下从map中读取
    return v, existed
}
func (m *RWMap) Set(k int, v int) { // 设置一个键值对
    m.Lock()              // 锁保护
    defer m.Unlock()
    m.m[k] = v
}
func (m *RWMap) Delete(k int) { //删除一个键
    m.Lock()                   // 锁保护
    defer m.Unlock()
    delete(m.m, k)
}
func (m *RWMap) Len() int { // map的长度
    m.RLock()   // 锁保护
    defer m.RUnlock()
    return len(m.m)
}
func (m *RWMap) Each(f func(k, v int) bool) { // 遍历map
    m.RLock()             //遍历期间一直持有读锁
    defer m.RUnlock()
    for k, v := range m.m {
        if !f(k, v) {
            return
        }
    }
}
```

## 分片锁实现：