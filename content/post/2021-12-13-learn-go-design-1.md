---
title: Go设计篇-1-创建Http服务
date: '2021-12-13'
slug: learn-go-design-1
categories:
  - programing
tags:
  - go
---

## 1 http客户端

`http.Client`是一个结构体类型，并且它包含的字段都是公开的。之所以该类型的零值仍然可用，是因为它的这些字段要么存在着相应的缺省值，要么其零值直接就可以使用，且代表着特定的含义。

```go
url1 := "http://google.cn"
fmt.Printf("Send request to %q with method GET ...\n", url1)
resp1, err := http.Get(url1)
if err != nil {
	fmt.Printf("request sending error: %v\n", err)
}
defer resp1.Body.Close()
line1 := resp1.Proto + " " + resp1.Status
fmt.Printf("The first line of response:\n%s\n", line1)
```

## 2 http服务端

`ListenAndServe`处理过程：

> 1.  检查当前的`http.Server`类型的值（以下简称当前值）的`Addr`字段。该字段的值代表了当前的网络服务需要使用的网络地址，即：IP 地址和端口号. 如果这个字段的值为空字符串，那么就用`":http"`代替。也就是说，使用任何可以代表本机的域名和 IP 地址，并且端口号为`80`。
> 2.  通过调用`net.Listen`函数在已确定的网络地址上启动基于 TCP 协议的监听。
> 3.  检查`net.Listen`函数返回的错误值。如果该错误值不为`nil`，那么就直接返回该值。否则，通过调用当前值的`Serve`方法准备接受和处理将要到来的 HTTP 请求。
>

`net.Listen`处理过程：

> 1.  解析参数值中包含的网络地址隐含的 IP 地址和端口号；
> 2.  根据给定的网络协议，确定监听的方法，并开始进行监听。
>

`http.Server`类型的`Serve`方法是怎样接受和处理 HTTP 请求的。

> 1. 在一个`for`循环中，网络监听器的`Accept`方法会被不断地调用，该方法会返回两个结果值；
> 2. 第一个结果值是`net.Conn`类型的，它会代表包含了新到来的 HTTP 请求的网络连接；第二个结果值是代表了可能发生的错误的`error`类型值。

**原生的创建方式**

```go
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello World!")
	})
	http.HandleFunc("/time/", func(w http.ResponseWriter, r *http.Request) {
		t := time.Now()
		timeStr := fmt.Sprintf("{\"time\": \"%s\"}", t)
		w.Write([]byte(timeStr))
	})

	http.ListenAndServe(":8080", nil)
}

```

**使用httprouter做简单的路由**

```go
func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
	fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
}

func main() {
	router := httprouter.New()
	router.GET("/", Index)
	router.GET("/hello/:name", Hello)

	log.Fatal(http.ListenAndServe(":8080", router))
}

```

