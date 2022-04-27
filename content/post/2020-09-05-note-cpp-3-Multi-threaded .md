---
title: 笔记-C++(3)-smulti-threaded开发
date: '2020-09-05'
slug: note-cpp-3-multi-threaded
categories:
  - problems
tags:
  - cpp
---

在 C++ 里，有四个基本线程处理的工具：仅调用一次、线程局部存储、原子变量和线程对象。

## 仅调用1次 

先声明一个 once_flag 类型的变量，最好是静态、全局的（线程可见），作为初始化的标志：

```
static std::once_flag flag; // 全局的初始化标志
```

然后调用专门的 call_once() 函数，以函数式编程的方式，传递这个标志和初始化函数。这样 C++ 就会保证，即使多个线程重入 call_once()，也只能有一个线程会成功运行初始化。

```

auto f = []() // 在线程里运行的lambda表达式
{
std::call_once(flag, // 仅一次调用，注意要传flag
[](){ // 匿名lambda，初始化函数，只会执行一次
cout << "only once" << endl;
} // 匿名lambda结束
); // 在线程里运行的lambda表达式结束
};

thread t1(f); // 启动两个线程，运行函数f
thread t2(f);
```


## 线程局部存储
线程局部存储这个功能在 C++ 里由关键字 thread_local 实现，它是一个和 static、extern 同级的变量存储说明，有 thread_local 标记的变量在每个线程里都会有一个独立的副本，是“线程独占”的，所以就不会有竞争读写的问题。

示例：
```
thread_local int n = 0; // 线程局部存储变量
auto f = [&](int x) // 在线程里运行的lambda表达式，捕获引用
{
	n += x; // 使用线程局部变量，互不影响
	cout << n; // 输出，验证结果
};
thread t1(f, 10); // 启动两个线程，运行函数f
thread t2(f, 20);
```
先定义了一个线程独占变量，然后用 lambda 表达式捕获引用，再放进多个线程里运行。


## 原子变量
对于那些非独占、必须共享的数据.要想保证多线程读写共享数据的一致性，关键是要解决同步问题，不能让两个线程同时写，也就是“互斥”。
这在多线程编程里早就有解决方案了，就是互斥量（Mutex）。但它的成本太高，所以，对于小数据，应该采用“原子化”这个更好的方案。

所谓原子（atomic），在多线程领域里的意思就是不可分的。操作要么完成，要么未完成，不能被任何外部操作打断，总是有一个确定的、完整的状态。所以也就不会存在竞争读写的问题，不需要使用互斥量来同步，成本也就更低。但不是所有的操作都可以原子化的，否则多线程编程就太轻松了。目前，C++ 只能让一些最基本的类型原子化，比如 atomic_int、atomic_long，等等：

```
using atomic_bool = std::atomic<bool>; // 原子化的bool
using atomic_int = std::atomic<int>; // 原子化的int
using atomic_long = std::atomic<long>; // 原子化的long
```
原子变量禁用了拷贝构造函数，所以在初始化的时候不能用“=”的赋值形式，只能用圆括号或者花括号

```
atomic_int x {0}; // 初始化，不能用=
atomic_long y {1000L}; // 初始化，只能用圆括号或者花括号
assert(++x == 1); // 自增运算
y += 200; // 加法运算
assert(y < 2000); // 比较运算
```
原子变量的主要应用场景：
- 当作线程安全的全局计数器或者标志位
- 实现高效的无锁数据结构（lock-free）。

## 线程对象
C++ 标准库的`std::this_thread` 里，有 `yield()`、`get_id()`、`sleep_for()`、`sleep_until()` 等线程管理函数。
用法示例：

```
static atomic_flag flag{false}; // 原子化的标志量
static atomic_int n;            // 原子化的int
auto f = [&]()                  // 在线程里运行的lambda表达式，捕获引用
{
    auto value = flag.test_and_set(); // TAS检查原子标志量
    if (value)
    {
        cout << "flag has been set." << endl;
    }
    else
    {
        cout << "set flag by " << this_thread::get_id() << endl; // 输出线程id
    }
    n += 100;               // 原子变量加法运算
    this_thread::sleep_for( // 线程睡眠
        n.load() * 10ms);   // 使用时间字面量
    cout << n << endl;
};            // 在线程里运行的lambda表达式结束
thread t1(f); // 启动两个线程，运行函数f
thread t2(f);
t1.join(); // 等待线程结束
t2.join();
```

建议使用**异步运行**的思路，调用函数 `async()`，“异步运行”一个任务，隐含的动作是启动一个线程去执行，但不绝对保证立即启动（也可以在第一个参数传递 `std::launch::async`，要求立即启动线程）。大多数 thread 能做的事情也可以用 `async()` 来实现，但不会看到明显的线程：


其实，这还是函数式编程的思路，在更高的抽象级别上去看待问题，异步并发多个任务，让底层去自动管理线程，要比我们自己手动控制更好（比如内部使用线程池或者其他机制）。`sync()` 会返回一个 `future `变量，可以认为是代表了执行结果的“期货”，如果任务有返回值，就可以用成员函数 `et() `获取。
不过要特别注意，`get() `只能调一次，再次获取结果会发生错误，抛出异常`std::future_error`。（至于为什么这么设计我也不太清楚，没找到官方的解释）另外，这里还有一个很隐蔽的“坑”，如果你不显式获取 `async()` 的返回值（即 `future `对象），它就会同步阻塞直至任务完成（由于临时对象的析构函数），于是`async`就变成了`sync`。所以，即使我们不关心返回值，也总要用` auto `来配合 `async()`，避免同步阻塞。

异步运行代码示例：

```
auto task = [](auto x) // 在线程里运行的lambda表达式
{
	this_thread::sleep_for(x * 1ms); // 线程睡眠
	cout << "sleep for " << x << endl;
	return x;
};
auto f = std::async(task, 10); // 启动一个异步任务
f.wait(); // 等待任务完成
assert(f.valid()); // 确实已经完成了任务
cout << f.get() << endl; // 获取任务的执行结果
```

