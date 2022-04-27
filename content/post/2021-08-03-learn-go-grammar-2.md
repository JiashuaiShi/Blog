---
title: Go语法篇-2-测试编码规范
date: '2021-08-03'
slug: learn-go-grammar-2
categories:
  - programing
tags:
  - go
---

## 1-单元测试：

Go 语言推荐测试文件和源代码文件放在一块，测试文件以 `_test.go` 结尾。

```
example/
   |--calc.go
   |--calc_test.go
```

- 测试用例失败：t.Error/t.Errorf  试用例终止：t.Fatal/t.Fatalf

- 测试用例名称一般命名为 `Test` 加上待测试的方法名。

- 测试用的参数有且只有一个，在这里是 `t *testing.T`。基准测试(benchmark)的参数是 `*testing.B`，TestMain 的参数是 `*testing.M` 类型。

- 执行测试用例：go test , 显示详细信息 -v。 `-cover` 参数可以查看覆盖率。

- http开发时，使用标准库`net/http/httptest`。


表驱动的测试用例的写法：

  ```go
  //  calc_test.go
  func TestMul(t *testing.T) {
  	cases := []struct {
  		Name           string
  		A, B, Expected int
  	}{
  		{"pos", 2, 3, 6},
  		{"neg", 2, -3, -6},
  		{"zero", 2, 0, 0},
  	}
  
  	for _, c := range cases {
  		t.Run(c.Name, func(t *testing.T) {
  			if ans := Mul(c.A, c.B); ans != c.Expected {
  				t.Fatalf("%d * %d expected %d, but %d got",
  					c.A, c.B, c.Expected, ans)
  			}
  		})
  	}
  }
  ```

  

## 2-基准压测：

- 函数名必须以 `Benchmark` 开头，后面一般跟待测试的函数名
- 参数为 `b *testing.B`。
- 执行基准测试时，需要添加 `-bench` 参数。

一个基准测试的例子：

```go
func BenchmarkHello(b *testing.B) {
    for i := 0; i < b.N; i++ {
        fmt.Sprintf("hello")
    }
}
```

使用 `RunParallel` 测试并发性能。

```go
func BenchmarkParallel(b *testing.B) {
	templ := template.Must(template.New("test").Parse("Hello, {{.}}!"))
	b.RunParallel(func(pb *testing.PB) {
		var buf bytes.Buffer
		for pb.Next() {
			// 所有 goroutine 一起，循环一共执行 b.N 次
			buf.Reset()
			templ.Execute(&buf, "World")
		}
	})
}
```

## 3-测试框架

常用GoConvey测试框架：

- 兼容go原始的测试语法
- 输出报告友好，支持webUI测试
- 用例可通过WebUI生成
