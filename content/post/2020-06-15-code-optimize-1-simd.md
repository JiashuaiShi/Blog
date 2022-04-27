---
title: 代码优化(1)—SIMD优化
date: '2020-06-15'
slug: code-optimize-1-simd
categories:
  - programing
tags:
  - optimize
---
simd优化是使用Intel或者编译期厂商提供的simd库，来对代码中并行计算的部分，进行优化的一种手段。全称Single Instruction Multiple Data，单指令多数据流。是一种采用一个控制器来控制多个处理器，同时对一组数据（又称“数据向量”）中的每一个分别执行相同的操作从而实现空间上的并行性的技术。简单而言，是指一条指令能够处理多个数据。

比如，在图像处理过程中，由于图像的数据常用的数据类型是RGB565, RGBA8888, YUV422等格式，这些格式的数据特点是一个像素点的一个分量总是用小于等于８bit的数据表示的。如果使用传统的处理器做计算，虽然处理器的寄存器是32位或是64位的，处理这些数据却只能用于他们的低８位，效率太低。如果把64位寄存器拆成８个８位寄存器就能同时完成８个操作，计算效率提升了８倍。这就是SIMD指令的核心思想。


## 字符串比较
原始版本：使用for循环进行比较

```
for i, _ := range a {
    if a[i] != b[i] { //按顺序比较数组a和数组b中的每个byte
```
SIMD优化：通过单指令同时处理多个byte数据的方式，大幅减少数据加载和比较操作的指令条数，发挥SIMD运算部件的算力，提升性能。
对字符数组采用分块的思路，将数组划分为64/16/8/4/2/1bytes大小的块，使用多个向量寄存器，利用一条SIMD指令最多同时处理16个bytes的优势，同时也减少了边界检查的次数。



## 向量内积


### 原始向量

运算前字节对齐-windows平台
```
__declspec(align(16)) float Arr[] = {1.0f, 2.0f, 3.0f, 4.0f};
```
运算前字节对齐-linux平台

```
__attribute__((aligned(16))) float Arr[] = {1.0f, 2.0f, 3.0f, 4.0f};
```

### 求内积函数
```
#include <stdint.h>

float vector_dot(uint32_t len,const float* a,const float* b)
{
    double sum=0;
    for(uint32_t i=0;i<len;i++)
        sum+=a[i]*b[i];
    return (float)sum;
}
```

在这种实现中，各个分量的相乘是串行的。这种串行方式叫做SISD，也就是Single Instruction Single Data，即一条指令处理一条数据。
Intel SSE指令集能够把4个分量各自的乘法运算同时进行，最后在把各个分量的乘积相加，即可以把四组单精度浮点数同时运算,理论上性能可以提高4倍。

### SEE优化后函数
```
#include <stdint.h>
#include <x86intrin.h>

float vector_dot(uint32_t len,const float* a,const float* b)
{
    double sum=0;
    uint32_t i=0;
    uint32_t end=len&(~3);
    for(;i<end;i+=4)
    {
        __m128 A=_mm_load_ps(a+i);
        __m128 B=_mm_load_ps(b+i);
        __m128 C=_mm_dp_ps(A,B,0xf1);
        sum+=C[0];
    }
    for(;i<len;i++)
        sum+=a[i]*b[i];
    return (float)sum;
}
```

使用`-msse4.1`编译选项编译。
```
gcc -msse4.1 test.c -o test
```


`__attribute__((aligned(16)))`对应`__m128`这个数据结构指向的地址必须是`128-bit aligne`d的，也就是16字节对齐。
代码中的第一个for循环中，把4个分量作为一个LOOP地处理，通过`_mm_dp_ps`来计算四对float的点乘，然后把所有的点乘结果累计。由于`len`可能不能被4整除，所以最后再用本来的办法把剩余的几个分量累加。这里需要说明的是_`mm_dp_ps`的第三个参数mask。第三个参数mask的高4位用来指明哪些分量参与计算。比如我只想计算第0,2,3这3个分量的内积，那么高4位就是1101。由于我要4个分量都参与计算，所以就是1111。低4位表面计算结果放到哪些位置。由于计算结果也是一个__m128类型，即也是4个float，那么就必须说明结果存放在哪个float中。我想放在第0个float，那么就是0001，所以最终`mask`就是`11110001`,即0xf1。


### AVX优化后函数
```
#include <stdint.h>
#include <x86intrin.h>

float vector_dot(uint32_t len,const float* a,const float* b)
{
    double sum=0;
    uint32_t i=0;
    uint32_t end;
    end=len&(~7);
    for(;i<end;i+=8)
    {
        __m256 tmp=_mm256_dp_ps(_mm256_load_ps(a+i),_mm256_load_ps(b+i),0xf1);
        sum+=tmp[0]+tmp[4];
    }
    end=len&(~3);
    for(;i<end;i+=4)
    {
        __m128 C=_mm_dp_ps(_mm_load_ps(a+i),_mm_load_ps(b+i),0xf1);
        sum+=C[0];
    }
    for(;i<len;i++)
        sum+=a[i]*b[i];
    return (float)sum;
}
```
通过`gcc -mavx test.c -o test` 编译。
使用256位寄存器，相比128位寄存器性能提升一倍。

Tips：使用_mm_loadu_ps代替_mm_load_ps解决字节对齐。

## 优点
- 效率高：单指令多数据流意味着只需要一个指令周期就能同时对多个数据进行批处理，虽然该类指令本身的指令周期可能会较一般的指令长，但是整体考虑肯定是提高了处理效率。
- 针对浮点数优化明显。

## 缺点

- 适用场景有限：并不是所有的情况都能使用SIMD，有些情况下就算能使用，也需要对原有算法进行不小的改动。
- 人为编写：目前编译器对SIMD的翻译有限，使用时需要开发者人为编写。
。
