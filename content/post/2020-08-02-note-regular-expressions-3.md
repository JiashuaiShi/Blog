---
title: 正则表达式(3)-C++正则表达式库
date: '2020-08-02'
slug: note-regular-expressions-3
categories:
  - problems
tags:
  - use
---


C++ 正则表达式主要有两个类。
- `regex`：表示一个正则表达式，是 `basic_regex `的特化形式；
- `smatch`：表示正则表达式的匹配结果，是 `match_results `的特化形式。

C++ 正则匹配有三个算法
- `regex_match()`：完全匹配一个字符串；
- `regex_search()`：在字符串里查找一个正则匹配；
- `regex_replace()`：正则查找再做替换。

Tips：
- 注意它们都是“只读”的，不会变动原字符串。
- 建议使用“原始字符串”。

生成正则表达式
```
static
auto make_regex = [](const auto& txt)
{
    return std::regex(txt);
};

static
auto make_match = []()
{
    return std::smatch();
};
```
正则匹配：

```
void test()
{
    auto str = "neir:automata"s;

    auto reg  = make_regex(R"(^(\w+)\:(\w+)$)"); // 原始字符串定义正则表达式
    auto what = make_match();

    assert(regex_match(str, what, reg));

    for(const auto& x : what) {
        cout << x << ',';
    }
    cout << endl;
}
```
定义两个lambda 表达式，生产正则对象，主要是为了方便用 auto自动类型推导。当然，同时也隐藏了具体的类型信息，将来可以随时变化。
然后我们就可以调用 regex_match() 检查字符串，函数会返回 bool 值表示是否完全匹配正则。如果匹配成功，结果存储在 what 里，可以像容器那样去访问，第 0 号元素是整个匹配串，其他的是子表达式匹配串。
