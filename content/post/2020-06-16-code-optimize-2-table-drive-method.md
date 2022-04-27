---
title: 代码优化(2)—表驱动法
date: '2020-06-16'
slug: code-optimize-2-table-drive-method
categories:
  - problems
tags:
  - optimize
---

《代码大全》提供了一种优化的方法——表驱动法。表驱动是一种编程模式，核心在于将输入变量作为直接或者间接索引到表里面查找直接的结果或者处理函数，索引表可以是一个数组、map、或者其它数据结构。

比如查找一年中每个月份的天数，如果用表驱动法，完全不需要写一堆if…else…语句，直接把每个月份的天数存到一个数组里就行了，取值的时候直接下标访问，最多针对二月判断一下闰年。这么算的话，平时用的的HashMap等也可以算是表驱动。

表里可以存数据，也可以存指令，或函数指针等都可以。
- 可读性强，数据处理流程一目了然。
- 便于维护，只需要增、删数据索引和方法就可以实现功能。
- 精简代码，降低圈复杂度。减少if else、switch、case使用。

## 一个计算器的程序
设计一个加减乘除的程序，用两种实现方式分别实现，进行对比：


### 常规方法：
通过switch、case处理多条件场景。
```
int calculatorData(char symbol, int a, int b)
{
	int ret = 0;
	switch (symbol)
	{
	case '+':
		ret = add(a, b);
		break;
	case '-':
		ret = subtraction(a, b);
		break;
	case '*':
		ret = multiplication(a, b);
		break;
	case '/':
		ret = division(a, b);
		break;
	default:
		break;
	}
	return ret;
}
```

### 表驱动法：
表驱动实现方法，结合函数指针数组处理多条件场景。通过表驱动来获取相对应的函数指针，从而避免了大量switch case的判断，起到了代码优化的效果。
![表驱动法示意图](http://qo1qkz2df.hn-bkt.clouddn.com/table-drived.png)
```
void InitTableFunc(FUNC* tableFunc) // 初始化函数指针
{
tableFunc['+'] = add;
tableFunc['-'] = subtraction;
tableFunc['*'] = multiplication;
tableFunc['/'] = division;
}

int tableCalculatorData(FUNC* tableFunc, char symbol, int a, int b)
{
if(symbol != '+' && symbol != '-' && symbol != '*' && symbol != '/')
{
return 0;
}
return (*tableFunc[symbol])(a, b);
}
```
### 测试程序：

公共基础方法
```
int add(int a, int b)
{
	return a + b;
}
int subtraction(int a, int b)
{
	return a - b;
}
int multiplication(int a, int b)
{
	return a * b;
}
int division(int a, int b)
{
	return a / b;
}
```
测试代码：
```
int main()
{
	const int N = 1000000;
	FUNC tableFunc[100];
	InitTableFunc(tableFunc);

	int res = 0;
	for (int i = 0; i < N; ++i)
	{
 		//res = calculatorData('+', res, i);
		res = tableCalculatorData(tableFunc, '+', res, i);
	}
	return 0;
}
```
参考资源：
[表驱动优化代码](https://www.smiletoyou.cn/?p=231 "blog")
https://www.smiletoyou.cn/?p=231

