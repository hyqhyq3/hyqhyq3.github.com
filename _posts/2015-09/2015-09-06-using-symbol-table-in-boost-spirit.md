---
layout: post
title: 在Spirit中使用符号表
category: 技术
tags: C++ Boost Spirit
---

在写解析器的时候，会碰到一些简单的符号的处理，比如"a>1", "b<=2"，如果为每个比较操作符都写一条rule，会感觉有点傻。

在Spirit中，可以用symbols来干这个事情。定义一些字符串到另一个类型的映射

```c++
// 代码抄袭自boost文档
using boost::spirit::qi::symbols;

symbols<char, int> sym;

sym.add
    ("Apple", 1)
    ("Banana", 2)
    ("Orange", 3)
;

int i;
test_parser_attr("Banana", sym, i);
std::cout << i << std::endl;
```

这样很简单地将代表水果的字符串解析成1，2，3这些数字。