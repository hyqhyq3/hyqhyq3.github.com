---
layout: post
title: boost.spirit入门(2)
category: 技术
tags: boost spirit c++
---

上一篇文章实现了一个解析`1, 2, 3, 4`字符串，这篇文章稍微复杂点，来解析`a=1,b=2,c=3`，将其存储到一个std::map\<std::string, int\>变量中。

通过查看文档，很容易写出一个EBNF(for spirit)， `(+(qi::char_ - '=') >> qi::lit('=') >> qi::int_ ) % qi::lit(',')`。
可是在存储到map中的时候出了点问题，怎么把一个`std::vector<std::tuple<std::string, int>>`的类型转换到我们需要的map类型。

spirit的作者老早就解决了这个问题，他发明了一个fusion库，fusion库可以让一个结构体像tuple一样，通过索引访问。当然也可以把pair当成tuple访问。

```cpp
#include <iostream>
#include <map>
#include <boost/spirit/include/qi.hpp>
#include <boost/fusion/adapted/std_pair.hpp>
using namespace boost::spirit;

int main()
{
	std::string str = "a=1, b=2, c=3";
	std::map<std::string, int> result;
	auto first = str.begin();
	auto last = str.end();
	qi::phrase_parse(first, last, ( +(qi::char_ - '=') >> qi::lit('=') >> qi::int_ )  % qi::lit(','), ascii::space, result);
	if(first == last)
	{
		std::cout << "success" << std::endl;
		for(auto i : result)
		{
			std::cout << "key = " << i.first << ", value = " << i.second << std::endl;
		}
	}
	else
	{
		std::cout << "error at " << std::string(first, last) << std::endl;
	}
}
```

这里用到的新东西不多， `qi::char_ - '='`表示的就是正则表达式里面的`[^=]`，从字符集里面排出一部分字符，用减号还是很形象的。