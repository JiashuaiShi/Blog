---
title: 笔记-C++(2)-smart pointer用法
date: '2020-09-04'
slug: note-cpp-2-smart-pointer
categories:
  - problems
tags:
  - cpp
---

智能指针是代理模式的具体应用，它使用 RAII 技术代理了裸指针，能够自动释放内存，无需程序员干预，所以被称为“智能指针”

## unique_ptr
### 理解含义
- 实际上并不是指针，而是一个对象。所以，不要企图对它调用 delete，它会自动管理初始化时的指针，在离开作用域时析构释放内存。
- 它也没有定义加减运算，不能随意移动指针地址，这就完全避免了指针越界等危险操作，可以让代码更安全
- 表示指针的所有权是“唯一”的，不允许共享，任何时候只能有一个“人”持有它。为了实现这个目的，unique_ptr 应用了 C++ 的“转移”（move）语义，同时禁止了拷贝赋值，所以，在向另一个 unique_ptr 赋值的时候，要特别留意，必须用 std::move() 函数显式地声明所有权转移。赋值操作之后，指针的所有权就被转走了，原来的 unique_ptr 变成了空指针，新的unique_ptr 接替了管理权，保证所有权的唯一性。

### 使用示例：
对象构造：

```
unique_ptr<string> ptr2(new string("hello")); // string智能指针
unique_ptr<int> ptr1(new int(10)); // int智能指针
```
工厂函数创建：
```
auto ptr3 = make_unique<int>(42); // 工厂函数创建智能指针
assert(ptr3 && *ptr3 == 42);
auto ptr4 = make_unique<string>("god of war"); // 工厂函数创建智能指针
assert(!ptr4->empty())
```

转移语义，保证unique

```
template<class T, class... Args> // 可变参数模板
std::unique_ptr<T> // 返回智能指针
my_make_unique(Args&&... args) // 可变参数模板的入口参数
{
	return std::unique_ptr<T>( new T(std::forward<Args>(args)...)); // 完美转发
}
```
## shared_ptr
### 理解含义

- 但 shared_ptr 的名字明显表示了它与 unique_ptr 的最大不同点：它的所有权是可以被安全共享的，也就是说支持拷贝赋值，允许被多个“人”同时持有，就像原始指针一样。
- shared_ptr 支持安全共享的秘密在于内部使用了“引用计数，引用计数的存储和管理都是成本
- 如果发生拷贝赋值——也就是共享的时候，引用计数就增加，而发生析构销毁的时候，引用计数就减少。只有当引用计数减少到0，也就是说，没有任何人使用这个指针的时候，它才会真正调用 delete 释放内存。
- shared_ptr 的引用计数也导致了一个新的问题，就是“循环引用。

### 使用示例
```
shared_ptr<int> ptr1(new int(10)); // int智能指针
assert(*ptr1 = 10); // 可以使用*取内容
shared_ptr<string> ptr2(new string("hello")); // string智能指针
assert(*ptr2 == "hello"); // 可以使用*取内容
auto ptr3 = make_shared<int>(42); // 工厂函数创建智能指针
assert(ptr3 && *ptr3 == 42); // 可以判断是否为空指针
auto ptr4 = make_shared<string>("zelda"); // 工厂函数创建智能指针
assert(!ptr4->empty()); // 可以使用->调用成员函数
```

## weak_ptr。

### 理解含义
- weak_ptr 顾名思义，功能很“弱”。它专门为打破循环引用而设计，只观察指针，不会增加引用计数（弱引用），但在需要的时候，可以调用成员函数 lock()，获取shared_ptr（强引用）。

### 使用示例

```
class Node final
{
public:
	using this_type = Node;
	using shared_type = std::weak_ptr<this_type>;
public:
	shared_type next; 
};

auto n1 = make_shared<Node>(); // 工厂函数创建智能指针
auto n2 = make_shared<Node>(); // 工厂函数创建智能指针
n1->next = n2; // 两个节点互指，形成了循环引用
n2->next = n1;
assert(n1.use_count() == 1); // 因为使用了weak_ptr，引用计数为1
assert(n2.use_count() == 1); // 打破循环引用，不会导致内存泄漏
if (!n1->next.expired()) { // 检查指针是否有效
	auto ptr = n1->next.lock(); // lock()获取shared_ptr
	assert(ptr == n2);
}
```

