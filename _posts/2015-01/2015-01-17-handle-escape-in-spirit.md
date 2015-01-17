---
layout: post
title: 使用spirit处理转移符
category: 技术
tags: boost spirit
---

```cpp
// rule<Iterator, std::string()>
mystr = (qi::char_ | escape_char )*
// rule<Iterator, char()>
escape_char = qi::lit('%') >> uint_parser<char, 16, 2, 2>();
```

这里可以将%20解析成' '，%3B解析成';'

[文档链接](http://www.boost.org/doc/libs/1_57_0/libs/spirit/doc/html/spirit/qi/reference/numeric/uint.html)