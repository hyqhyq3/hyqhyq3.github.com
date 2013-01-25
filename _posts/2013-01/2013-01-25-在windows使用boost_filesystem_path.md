---
layout: post
title: 在windows使用boost::filesystem::path
category: cpp
tags: boost cpp
---

{% highlight cpp %}
boost::filesystem::path config_file("config");
po::store ( po::parse_config_file<char> ( config_file.c_str(), desc ), vm ); // 这里将会报错,不能正常编译.
{% endhighlight %}

查看过boost文档后,发现windows,mingw等环境下,value_type是wchar_t, 那么path::c_str()返回的类型是const value_type*也就是 const wchat_t*,
没有办法使用program_options.

继续看文档,发现有一个path::string(), 将代码改成如下

{% highlight cpp %}
boost::filesystem::path config_file("config");
po::store ( po::parse_config_file<char> ( config_file.string().c_str(), desc ), vm ); // OK now.
{% endhighlight %}

这时候可以正常工作了.
最开始是因为觉得这样子写太麻烦, 就直接用了c_str(). 没想到在linux上ok, 到windows上却出问题.
