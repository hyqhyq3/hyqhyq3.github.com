---
layout: post
title: boost.spirit入门(1)
category: 技术
tags: boost spirit c++
---

boost.spirit是一个parser/generator generator，就是用来写解析器和生成器的工具。在写小型解析器的时候，boost.spirit比flex和yacc方便很多。

boost::spirit::qi名字空间内，是parser相关的库。
这篇文章主要是讲了如何实现一个数字串程序，会用到一些基础的qi的功能，parser api， attr等。

最开始来个简单的热身，如何解析一串数字

	1, 2, 3, 4, 5,

我们将把他存入一个std::vector<int>中。
使用spirit的attr，可以非常快速地实现这样的功能。

> attr就是指一个文本，比如说*123*，代表的一个东西，我们可以把它当成数字解析，那么它的attr就是123这个数字。
> 我们也可以把它当成字符串解析，那么它的attr就是"123"这个字符串

```c
#include <iostream>
#include <boost/spirit/include/qi.hpp>
using namespace boost::spirit;

int main()
{
	std::string str = "1, 2, 3, 4, 5,";
	std::vector<int> result;
	auto first = str.begin();
	auto last = str.end();
	qi::phrase_parse(first, last, *(qi::int_ >> qi::lit(',')), ascii::space, result);
	if(first == last)
	{
		std::cout << "success" << std::endl;
		for(auto i : result)
		{
			std::cout << "number " << i << std::endl;
		}
	}
	else
	{
		std::cout << "error at " << std::string(first, last) << std::endl;
	}
}

```

phrase\_parse的第一个和第二个参数，表示要解析的字符串的迭代器。
phrase\_parser会改变first的值，表示当前解析到哪一个字符。


第三个参数就是解析器的grammar，用的是EBNF语法（应该说是EBNF for spirit）。
这里的`*(qi::int_ >> qi::lit(','))`就是普通EBNF的`grammar ::= ( int ',' ) * `，由于c＋＋算符重载的限制，kleene闭包被写在了句子的前面。同样因为c++的限制，短语之间需要使用>>连接。[qi::int_的文档](http://www.boost.org/doc/libs/1_57_0/libs/spirit/doc/html/spirit/qi/quick_reference/qi_parsers/numeric.html) [*的文档](http://www.boost.org/doc/libs/1_57_0/libs/spirit/doc/html/spirit/qi/quick_reference/qi_parsers/operator.html)

第四个参数skipper表示可以忽略的字符，这里的ascii::space，表示空格，tab等都是可以忽略的。

第五个参数attr，需要传递一个变量，来存储解析的结果，这个变量和类型和grammar对应。
qi::int\_的attr是int，qi::lit的attr是空，*(qi::int\_ >> qi::lit(','))的attr就是int的数组，即std::vector\<int\>。


改进
=======
上面的例子中，结尾多出了一个逗号，按照我们的习惯，结尾是不需要逗号的。那么改一下grammar，马上解决这个问题。

```cpp
qi::phrase_parse(first, last, qi::int_ % qi::lit(','), ascii::space, result);
```

%表示用分隔符分隔开的列表 [%的文档在这](http://www.boost.org/doc/libs/1_57_0/libs/spirit/doc/html/spirit/qi/quick_reference/qi_parsers/operator.html)
