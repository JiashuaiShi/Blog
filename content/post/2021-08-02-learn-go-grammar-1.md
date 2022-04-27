---
title: Go语法篇-1-反射使用
date: '2021-08-02'
slug: learn-go-grammar-1
categories:
  - programing
tags:
  - go
---
## 1. 什么是反射

> 反射是指在程序运行期对程序本身进行访问和修改的能力。程序在编译时，变量被转换为内存地址，变量名不会被编译器写入到可执行部分。在运行程序时，程序无法获取自身的信息。
>
> 支持反射的语言可以在程序编译期将变量的反射信息，如字段名称、类型信息、结构体信息等整合到可执行文件中，并给程序提供接口访问反射信息，这样就可以在程序运行期获取类型的反射信息，并且有能力修改它们。
>
> Go程序在运行期使用reflect包访问程序的反射信息。

## 2.使用注意

- 编译期难以检查错误，延迟到运行期报错。
- 影响分析检查工具的准确性。
- 动态反射造成局部性能的损失。

## 2. 具体例子：

### 1. 类型判断：

```go
func CheckType(v interface{}) {
	t := reflect.TypeOf(v)
	switch t.Kind() {
	case reflect.Float32, reflect.Float64:
		fmt.Println("Float")
	case reflect.Int, reflect.Int32, reflect.Int64:
		fmt.Println("Integer")
	default:
		fmt.Println("Unknown", t)
	}
}
```

### 2. 递归比较：

```go
s1 := []int{1, 2, 3}
s2 := []int{1, 2, 3}
t.Log("s1 == s2?", reflect.DeepEqual(s1, s2))
```

### 3. 函数调用：

```go
type Employee struct {
	EmployeeID string
	Name       string `format:"normal"`
	Age        int
}

func (e *Employee) UpdateAge(newVal int) {
	e.Age = newVal
}

func TestInvokeByName(t *testing.T) {
	e := &Employee{"1", "Mike", 30}
    // 1-获取字段名
	t.Logf("Name: value(%[1]v), Type(%[1]T) ", reflect.ValueOf(*e).FieldByName("Name"))
	if nameField, ok := reflect.TypeOf(*e).FieldByName("Name"); ok {
		// 2-获取标签tag
        t.Log("Tag:format", nameField.Tag.Get("format"))
	}
    // 3-调用函数
	reflect.ValueOf(e).MethodByName("UpdateAge").Call([]reflect.Value{reflect.ValueOf(1)})
}
```

