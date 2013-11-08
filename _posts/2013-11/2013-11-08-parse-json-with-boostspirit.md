---
layout: post
title: 使用boost.spirit解析JSON
category: 技术
tags: boost spirit c++
---

本来今天是想用flex/yacc做一个json解析器，用于加载配置文件（其实在boost.property_tree里面已经有了json_parser），结果折腾了半天，发现flex/yacc这类工具生成的代码真心不适合c++。
于是又捡起了boost.spirit。

说起boost.spirit，不得不感叹一下作者的强大，居然能把算符重载，模板推导用到这种程度，估计连BS都不认得了。。。

现在暂时用boost::any来存储json数据，打算有时间的时候改一改，换成更容易查询的自定义类型。

```cpp
#include <iostream>
#include <boost/spirit/include/qi.hpp>
#include <boost/fusion/include/std_pair.hpp>
#include <boost/any.hpp>
#include <map>

namespace qi = boost::spirit::qi;
namespace ascii = boost::spirit::ascii;

template<typename Iterator>
struct json_grammar : qi::grammar<Iterator, boost::any(), ascii::space_type>
{
  json_grammar()
    : json_grammar::base_type(element)
  {
    number %= qi::double_;
    array %= qi::eps >> '[' >> element % ',' >> ']';
    string %= qi::lexeme['"' >> +(ascii::char_ - '"') >> '"'];
    key_value %= string >> ':' >> element;
    object %= qi::eps >> '{' >> key_value % ',' >> '}';
    element %= array | object | string | number;
  }

  qi::rule<Iterator, std::vector<boost::any>(), ascii::space_type> array;
  qi::rule<Iterator, std::pair<std::string, boost::any>(), ascii::space_type> key_value;
  qi::rule<Iterator, std::map<std::string, boost::any>(), ascii::space_type> object;
  qi::rule<Iterator, std::string(), ascii::space_type> string;
  qi::rule<Iterator, double(), ascii::space_type> number;
  qi::rule<Iterator, boost::any(), ascii::space_type> element;
};


int main(int argc, char **argv) {
  typedef json_grammar<std::string::const_iterator> Parser;
  Parser p;
  const std::string str = R"({"abc":"def"})";
  boost::any result;
  if(qi::phrase_parse(str.begin(), str.end(), p, ascii::space, result))
  {
    std::cout << "format ok" << std::endl;
    std::map<std::string, boost::any> p = boost::any_cast<std::map<std::string, boost::any>>(result);
    std::cout << boost::any_cast<std::string>(p["abc"]) << std::endl;
  }
  return 0;
}
```


