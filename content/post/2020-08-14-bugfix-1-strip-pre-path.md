---
title: 去除__FILE__前缀路径
date: '2020-08-14'
slug: record-bugfix-1-strip-pre-path
categories:
  - problems
tags:
  - tricks
---
在做一些日志输出的工作时，想要获取当前文件名，而不是冗长的文件路径。路径获取往往和各家os底层函数优化。C++/C标准中定义了一些预处理宏，可以帮助我们获取文件路径。我们希望能够在编译期而不是在运行期做这个事情，避免额外的性能消耗。同时希望有一些跨平台的解决方案。以下是一些思路:

## 方法1：运行期去除前缀
`__FILE__` 可以在编译期获取绝对路径，`__BASE_NAME__`可以获取相对路径，但是不具有跨平台性。
我们想通过编译期来解决这个问题，但是暂时未找到能支持的编译选项。似乎只能在运行期做路径切割。
1. 通过`__FILE__`预处理期间获得路径。
2. 定义`inline`函数或者宏处理路径。`strchr`是一个有帮助的函数，即在`str`找到`ch`的指针。

```
#define __FILENAME__ (strrchr(__FILE__, '/') ? strrchr(__FILE__, '/') + 1 : __FILE__)
```
## 方法2：编译器函数优化
使用`__builtin_strrchr`替换`strchr`。
GNU工具链提供了一些常见的编译器优化函数，可以在编译期进行函数调用。这样就可以把路径切换转换到编译期处理。
缺点是：需要特定的编译器支持，GNU需要检查下自己的编译器是否支持，或者查看[GNU对应版本的文档][1]。
```
#define __FILENAME__ (__builtin_strrchr(__FILE__, '/') ? __builtin_strrchr(__FILE__, '/') + 1 : __FILE__)
```

## 方法3：在构建脚本中处理

在编译脚本中定义预处理宏`__FILENAME__`, 使用`shell`去除路径后的文件名，在编译期将文件名复制给`__FILENAME__`宏。
- 使用cmake构建脚本
```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D__FILENAME__='\"$(subst ${CMAKE_SOURCE_DIR}/,,$(abspath $<))\"'")
```
```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D__FILENAME__='"$(notdir $<)"'")
```
- 在makefile中更改

```
CXX_FLAGS+=-D__FILENAME__='\"$(subst $(SOURCE_PREFIX)/,,$(abspath $<))\"'"
```

Tips：当定义`___FILENAME__`预处理器宏时，可能需要开启`-Wno-builtin-macro-redefined`编译选项。
```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-builtin-macro-redefined")
```

## 方法4.0：使用constexpr在编译期处理
C++11只允许在`constexpr`函数中使用`return-statement`，C++14取消了这个限制。

- C++11版本
```
constexpr const char* str_end(const char* str) {
	return *str ? str_end(str + 1) : str;
}
constexpr bool str_slant(const char* str) {
	return *str == '/' ? true : (*str ? str_slant(str + 1) : false);
}
constexpr const char* r_slant(const char* str) {
	return *str == '/' ? (str + 1) : r_slant(str - 1);
}
constexpr const char* file_name(const char* str) {
	return str_slant(str) ? r_slant(str_end(str)) : str;
}
```

- C++14版本
```
constexpr const char* file_name(const char* path) {
    const char* file = path;
    while (*path) {
        if (*path++ == '/') {
            file = path;
        }
    }
    return file;
}
```
- [一个神奇的宏][2]

定义`constexpr`函数：
```
using cstr = const char * const;

static constexpr cstr past_last_slash(cstr str, cstr last_slash)
{
    return
        *str == '\0' ? last_slash :
        *str == '/'  ? past_last_slash(str + 1, str + 1) :
                       past_last_slash(str + 1, last_slash);
}

static constexpr cstr past_last_slash(cstr str) 
{ 
    return past_last_slash(str, str);
}
```
然后定义一个方便的宏方法
```
#define __SHORT_FILE__ ({constexpr cstr sf__ {past_last_slash(__FILE__)}; sf__;})
```

## 方法4.1： 验证在编译期所做的操作

 编译源码

```
source file name is foo/foo1/foo2/foo3/foo4.cpp
useg++ -o foo.exe foo/foo1/foo2/foo3/foo4.cpp -std=c++11 --save-temps to compile this file.

```

反编译后查看汇编代码

```
.file   "foo4.cpp"
        .section        .rodata
.LC0:
        .string "foo/foo1/foo2/foo3/foo4.cpp"
        .text
        .globl  main
        .type   main, @function
main:
.LFB4:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        subq    $16, %rsp
        movq    $.LC0+19, -8(%rbp) 
        movl    $.LC0+19, %edi
        call    puts
        movl    $0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
.LFE4:
        .size   main, .-main
        .ident  "GCC: (Ubuntu 4.8.4-2ubuntu1~14.04.3) 4.8.4"
        .section        .note.GNU-stack,"",@progbits
```
`movl    $.LC0+19, %edi` 中`.LC0 + 19`直接是去掉路径前缀的[地址][3]。


[1]:https://www.keil.com/support/man/docs/ARMCC/armcc_chr1359125006834.htm
[2]:https://blog.galowicz.de/2016/02/20/short_file_macro/
[3]:https://stackoverflow.com/questions/31050113/how-to-extract-the-source-filename-without-path-and-suffix-at-compile-time
