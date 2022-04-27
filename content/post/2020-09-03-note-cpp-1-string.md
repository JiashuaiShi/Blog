---
title: 笔记-C++(1)-String用法
date: '2020-09-03'
slug: note-cpp-1-string
categories:
  - problems
tags:
  - cpp
---

## 字符串实现
string是模板类 basic_string 的特化形式。
```
using string = std::basic_string<char>;
```

## 区分string与char *
### 标准库string
```
/*为了避免与用户自定义字面量的冲突，后缀“s”不能直接使用，必须用 using 打开名字空间才行*/
using namespace std::literals::string_literals;
auto str = "std string"s; // 后缀s，表示是标准字符串，直接类型推导
```

### 原始字符串（Raw string literal）
```
auto str = R"(nier:automata)"; // 原始字符串：nier:automata
```

### 一个小问题：在原始字符串里面写引号 + 圆括号的形式该怎么办呢？
对于这个问题，C++ 也准备了应对的办法，就是在圆括号的两边加上最多 16 个字符的特别“界定符”（delimiter），这样就能够保证不与字符串内容发生冲突。

```
auto str5 = R"==(R"(xxx)")==";// 原样输出：R"(xxx)"
```

### C++17 string_view的简单实现

```
class my_string_view final // 简单的字符串视图类，示范实现
{
public:
	using this_type = my_string_view; // 各种内部类型定义
	using string_type = std::string;
	using string_ref_type = const std::string&;
	using char_ptr_type = const char*;
	using size_type = size_t;
private:
	char_ptr_type ptr = nullptr; // 字符串指针
	size_type len = 0; // 字符串长度
public:
	my_string_view() = default;
	~my_string_view() = default;
	my_string_view(string_ref_type str) noexcept
		: ptr(str.data()), len(str.length())
	{}
public:
	char_ptr_type data() const // 常函数，返回字符串指针
	{
		return ptr;
	}
	size_type size() const // 常函数，返回字符串长度
	{
		return len;
	}
};
```

