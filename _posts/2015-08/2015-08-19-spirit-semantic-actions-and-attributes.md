---
layout: post
title: 将boost.spirit的语义动作与属性结合使用
category: 
tags: c++ c++11 spirit
---

spirit用于处理解析结果的方式有语义动作（semantic action）和属性（attribute）两种。使用semantic action灵活，使用attribute方便快捷。但是某些时候，将两者结合起来用，既灵活，又快捷。

简单地说，就是在semantic action里面，用phoenix::bind，将\_1和\_val作为参数传进函数中，然后就可以按照自己想要的方式来处理attribute

```c++
#include <boost/spirit/include/qi.hpp>
#include <boost/fusion/adapted/struct.hpp>
#include <boost/phoenix/fusion/at.hpp>
#include <boost/phoenix/bind.hpp>
#include <iostream>

using namespace boost::spirit;
namespace phx = boost::phoenix;


void map_(qi::unused_type& v, QueryAction& attr)
{
	attr.mytype = "map";
	std::cout << "got map" << std::endl;
}

void array_(qi::unused_type& v, QueryAction& attr)
{
	attr.mytype = "array";
	std::cout << "got array" << std::endl;
}

void field_(std::vector<char>& v, QueryAction& attr)
{
	attr.mytype = "field";
	attr.field.assign(v.begin(), v.end());
	std::cout << "got field " << attr.field << std::endl;
}

std::vector<QueryAction> ParseQuery(const std::string& query)
{
	std::vector<QueryAction> ret;
	typedef std::string::const_iterator Iterator;
	auto first = query.cbegin();
	auto last = query.cend();
	std::cout << "query " << query << std::endl;
	qi::rule<Iterator, QueryAction()> r1 = qi::lit("{}")[phx::bind(&map_, _1, _val)] | qi::lit("[]")[phx::bind(&array_, _1, _val)] | (+(qi::char_("0-9a-zA-Z_")))[phx::bind(&field_, _1, _val)];
	qi::phrase_parse(first, last, *(r1), lit("."), ret);
	return ret;
}
```
