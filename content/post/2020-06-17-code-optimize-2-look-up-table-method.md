---
title: 代码优化(3)—查表法与位运算
date: '2020-06-17'
slug: code-optimize-2-look-up-table-method
categories:
  - problems
tags:
  - optimize
---
## 查表优化

查表法是指将计算的结果预先存储起来，在后序重复使用到这个结果时，不需要再次进行计算，可通过查表得出，节省了重复计算的时间。
在动态规划算法中，可以采用类似的**记忆化搜索**的方法，将递推的中间结果存储起来，每次递归计算时，先在表中进行搜索，若已经计算过，则从表中查找返回。

### 斐波那契数列计算

递归计算：


```
long factorial(int i)
{
    if (i == 0)
        return 1;
    else
        return i * factorial(i - 1);
}
```

查表优化：

```
static long factorial_table[] = {1， 1， 2， 6， 24， 120， 720 /* etc */};

long factorial(int i)
{
    return factorial_table[i];
}

```



## 位运算优化
一般对int型变量进行位运算。

### 求余运算
模运算 `a=a%8` 
位运算 `a=a&7`


### 奇偶判断
模运算 `if(x % 1 == 1)`
位运算 `if(x & 1)`

### 判断相等
整数比较 `if(a == b)`
位运算 `if(!(a ^b))`

### 交换两个整数
库函数 
```
swap(a, b);
```
位运算
```
a=a^b;
b=a^b;
a=a^b;
```

### 判断两个数是否同号
```
!((a^b)>>31);
```

### 求n的绝对值
```
(n^(n>>31))-(n>>31); 
```

### 判断一个整数n是否为2的幂 2^n：
```
 ((x&(x-1))==0)&&(x!=0);
```

### 计算整数x和y的平均值，可防止溢出
```
int average(int x, int y) //返回X,Y 的平均值 
{ 
　　return (x&y)+((x^y)>>1); 
}
```
