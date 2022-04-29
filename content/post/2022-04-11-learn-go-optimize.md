---
title: Go代码优化的小例子
date: '2022-04-10'
slug: learn-go-optimize
categories:
  - programing
tags:
  - go
---

##  string优化

string有一些特性：

- 默认值是空字符串 ""
- ⽤索引号访问某字节，如 s[i]。不能⽤序号获取字节元素指针， &s[i] ⾮法
-  不可变类型，⽆法修改字节数组。字节数组尾部不包含 NULL。

不可变的好处：

- 线程安全。
- 方便内存共享。

例子：

拼接字符串的方式会导致的string创建、销毁和内存分配。可以使用`strings.Builder`来优化。

- `strings.Builder` 底层使用一个 `buf []byte` 来存放字符串的内容，避免string 在构建过程中会不断的被销毁重建。

- 为了解决`bytes.Buffer.String()`存在的`[]byte -> string`类型转换和内存拷贝问题，这里使用了一个`unsafe.Pointer`的存指针转换操作，实现了直接将`buf  []byte`转换为 string类型，同时避免了内存充分配的问题。
- 如果能够在栈上完成的工作逃逸到了堆上，性能就大打折扣了。因此，`copyCheck` 加入了一行比较hack的代码来避免buf逃逸到堆上。

```go
func BenchmarkStringBuilder(b *testing.B) {  
    b.ResetTimer()  
    for idx := 0; idx < b.N; idx++ {  
        var builder strings.Builder  
        for i := 0; i < numbers; i++ {  
            builder.WriteString(strconv.Itoa(i))    
        }  
        _ = builder.String()  
    }  
    b.StopTimer()  
}  
```



## easyJson调优

避免json使用反射做动态解析，而是使用预生成文件的方式做优化。缺点是需要单独生成文件。

```go
func TestEasyJson(t *testing.T) {  
    e := Employee{}  
    e.UnmarshalJSON([]byte(jsonStr))  
    fmt.Println(e)  
    if v, err := e.MarshalJSON(); err != nil {  
        t.Error(err)  
    } else {  
        fmt.Println(string(v))  
    }  
}
```



## 并发Map读写

读多写少使用：sync.Map

读写相近使用：ConcurrentMap
