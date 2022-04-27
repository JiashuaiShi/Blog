---
title: 编程题目(2)—位运算
date: '2021-01-17'
slug: algo-problem-2-bit-operation
categories:
  - programing
tags:
  - algo
---

题目来自：[求最大数值][0]

```
编写一个方法，找出两个数字a和b中最大的那一个。
不得使用if-else或其他比较运算符。

示例：

输入： a = 1, b = 2
输出： 2
```
## 思路分析：

### [短路特性 + 位运算][1]
条件运算符是具有短路特性的，当前面条件不满足时，后面就不会继续运算。所有可以使用短路特性来代替`if`判断条件。
判断两个数的大小关系，先作差，然后判断最高位的符号的正负即可。位运算要考虑溢出的情况，所以需要考虑同号和异号两种情况。
```
int maximum(int a, int b) {
	int res = a;
	//异号处理
	(a ^ b) >> 31 && (a >> 31) && (res = b);
	//同号处理
	(-(a ^ b) >> 31) && ((a - b) >> 31) && (res = b);
	return res;
}
```

### [异或运算][2]
1. 首先我们目标是得到一个标志ans，当a大的时候ans为1，b大的时候ans为0。如果我们能得到ans，则最终的答案是 `ans * a + (ans ^ 1) * b`;这里异或1的操作，把0变成1，把1变成0。

2. 如何得到ans，容易想到的是拿到b-a的符号位，但整数溢出会导致符号位不可靠。所以我们还需要拿到a和b的符号位，如果a，b同号，则相减不会溢出，如果异号，则直接选择非负的那个（即符号位为0）的即可。注：32位整型的符号位可以通过将原数右移31位得到。

3. 以下记a的符号位为x，b的符号位为y，b-a的符号位为z。
得到下列真值表：

|  x   |  y   |  z   | ans  |
| :--: | :--: | :--: | :--: |
|  0   |  0   |  0   |  0   |
|  0   |  0   |  1   |  1   |
|  0   |  1   |  0   |  1   |
|  0   |  1   |  1   |  1   |
|  1   |  0   |  0   |  0   |
|  1   |  0   |  1   |  0   |
|  1   |  1   |  0   |  0   |
|  1   |  1   |  1   |  1   |

通过真值表，使用卡诺图求出ans的表达式。有点类似于数字电路的思想。
根据从真值表求表达式，以及卡诺图相关知识，可以得到公式：
`ans = (!x & y) | (!x & z) | (y & z);`
注：这里!代表逻辑非，代码实现中应当使用`(x ^ 1)`代替比较好。
因为只用到了非x，以下代码中的x直接就是这里的`(x ^ 1)`。

```
int maximum(int a, int b) {
	unsigned aa = a, bb = b;
	bool x = (aa >> 31) ^ 1, y = bb >> 31, z = (bb - aa) >> 31;
	bool ans = (x & y) | (x & z) | (y & z);
	return ans * a + (ans ^ 1) * b;
}
```

**另外一种方法：**
1. `int`计算溢出的情况可以转化为`long long`计算。
2. 取出符号位可以通过位移获得。

[实现如下][4]：
```
int maximum(int a, int b) {
	long k = (((long)a - (long)b) >> 63) & 1;
	return b * k + a * (k ^ 1);
}
```
**最好把输入的a和b强行转成无符号整型再计算，否则leetcode的编译器不让做越界的减法运算。**


### [数学方法][3]
利用数学上的公式：
```
max(a, b) = ((a + b) + abs(a - b)) / 2
```
为了回避abs，利用位运算实现绝对值功能:
也可以考虑先平方，再开方来求得绝对值。但是存在大数超出表示范围的情况，可能需要转成double值运算，或者使用数组来做大数乘法。

```
abs(var) = (var ^ (var >> 63)) - (var >> 63)
```
或者
```
long absolute(long a) {
	int flag = a >> 63; //正数flag = 0，负数flag = -1
	return (flag ^ a) - flag; //任何数与0异或值不变，任何数与-1异或等价于按位取反
}
```
[0]:https://leetcode-cn.com/problems/maximum-lcci/
[1]:https://leetcode-cn.com/problems/maximum-lcci/comments/408118
[2]:https://leetcode-cn.com/problems/maximum-lcci/solution/cjie-fa-bu-yong-64wei-zheng-xing-bu-yong-shu-xue-h/
[3]:https://leetcode-cn.com/problems/maximum-lcci/solution/ji-yu-wei-yun-suan-shi-xian-da-xiao-bi-jiao-by-dex/
[4]:https://leetcode-cn.com/problems/maximum-lcci/solution/gen-ju-ti-mu-de-ti-shi-wan-cheng-wei-yun-suan-qiu-/