---
title: Go语法篇-3-Goroutine的使用
date: '2021-08-04'
slug: learn-go-grammar-3
categories:
  - programing
tags:
  - go
---
Go 语言实现了基于 CSP（Communicating Sequential Processes）理论的并发方案。channel 是用于 Goroutine 间通信。



## 无缓冲通道

- 同步阻塞。

- 信号通知的例子：

  ```go
  type signal struct{}
  
  func worker() {
  	println("worker is working...")
  	time.Sleep(10 * time.Second)
  }
  func spawn(f func()) <-chan signal {
  	c := make(chan signal)
  	go func() {
  		println("worker start to work...")
  		f()
  		c <- signal{}
  	}()
  	return c
  }
  func main() {
  	println("start a worker...")
  	c := spawn(worker)
  	<-c
  	fmt.Println("worker work done!")
  }
  ```

  

## 有缓冲通道

- 异步非阻塞
- 消息队列的例子：

```go
var queue = make(chan int, 10)
var wg sync.WaitGroup

func producer() {
	for i := 1; i < 11; i++ {
		queue <- i
		fmt.Println("生产：", i)
		time.Sleep(time.Second * 2)
	}
	wg.Done()
}

func consumer() {
	for {
		select {
		case i := <-queue:
			fmt.Println("消费：", i)
			time.Sleep(time.Second)
		default:
			fmt.Println("消息队列为空")
		}
	}
}
```

## select结合使用

- default分支，避免阻塞

  ```go
  func trySend(c chan<- int, i int) bool {
  	select {
  	case c <- i:
  		return true
  	default: // channel满了
  		return false
  	}
  }
  ```

- 超时检测

  ```go
  func worker() {
  	select {
  	case <-c:
  		// ... do some stuff
  	case <-time.After(30 * time.Second):
  		return
  	}
  }
  ```

- 心跳机制

  ```go
  func worker() {
  	heartbeat := time.NewTicker(30 * time.Second)
  	defer heartbeat.Stop()
  	for {
  		select {
  		case <-c:
  			// ... do some stuff
  		case <-heartbeat.C:
  			//... do heartbeat stuff
  		}
  	}
  }
  ```

  