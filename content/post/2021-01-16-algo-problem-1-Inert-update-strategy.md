---
title: 编程题目(1)—惰性更新策略
date: '2021-01-16'
slug: Inert-update-strategy
categories:
  - programing
tags:
  - algo
---

题目来自[《程序员面试金典》栈排序][1]
## 题目描述：
```
编写程序，对栈进行排序使最小元素位于栈顶。最多只能使用一个其他的临时栈存放数据，但不得将元素复制到别的数据结构（如数组）中。
该栈支持如下操作：push、pop、peek 和 isEmpty。当栈为空时，peek 返回 -1。

示例1:

 输入：
["SortedStack", "push", "push", "peek", "pop", "peek"]
[[], [1], [2], [], [], []]
 输出：
[null,null,null,1,null,2]
示例2:

 输入： 
["SortedStack", "pop", "pop", "push", "pop", "isEmpty"]
[[], [], [], [1], [], []]
 输出：
[null,null,null,null,null,true]
说明:

栈中的元素数目在[0, 5000]范围内。
```
## 解题思路：
题目的描述是：实现一个API：`SortedStack`，称为有序栈，内部使用两个栈来进行实现。
一般的思路是：
1. `mainStk`维护一个有序的栈，每次`push`前，先把`mainStk`栈顶小于当前`val`的值移动到`helpStk`；
2. 把`val`值`push`到`mainStk`
3. 最后把helpStk中的元素push到mainStk中。

思考下是否可以继续优化，
当我们连续`push`某个值时，其实每次都需要把若干个元素暂时转移到`helpStack`中。
其实这个过程可以优化为，
当我们`push`这个值后，不要马上把`mainStack`恢复到有序的状态，而是根据下一个值的大小，来动态调整，使之`mainStack`继续满足可以`push`的状态。比如连续push相同值时，就可以在上个状态结束后，继续push。当我们想要取出最小值时，才将mainStack恢复到有序栈的状态。取出栈顶元素就是最小值。
这种方法其实是一种惰性更新的策略，也就是在真正需要的时候，才继续排序或者转移。类似于linux中的写时复制，在创建一个子进程时，没有马上复制父进程中的内存空间的数据。子进程在只读的情况下，可以沿用父进程中的内存数据。当子进程的内存空间的数据真正改变需要写入的时候，才进行复制。一般称为`copy-on-write`技术。这种技术提高了linux中进程创建的效率。

## 解题代码：
在`push`的时候，进行状态调整，使之满足使其满足 `helpStack.top() <= val <= mainStack.top()`,
然后`helpStack`和`mainStack`栈中的元素分别是有序的，`mainStack`栈中的元素是升序的，`helpStack`栈中的元素是降序的。此时，可以将`al`值push到mainStack中，则通过这两个栈维护的整个区间元素是有序的。
```
void push(int val) {
    while(st1.size() && st1.top() < val) {
        st2.push(st1.top());
        st1.pop();
    }

    while(st2.size() && st2.top() > val) {
        st1.push(st2.top());
        st2.pop();
    }

    st1.push(val);
}
```
当`peek`时，将`helpStack`元素`push`到`mainStack`中，`mainStack`是有序的，返回栈顶元素即可。


```
    int peek() {
       while(st2.size()) {
            st1.push(st2.top());
            st2.pop();
        }

        if(st1.size()) {
            return st1.top();
        }

        return -1;
    }
```
[1]:(https://leetcode-cn.com/problems/sort-of-stacks-lcci/)