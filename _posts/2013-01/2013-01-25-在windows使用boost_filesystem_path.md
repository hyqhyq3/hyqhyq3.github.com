---
layout: post
title: 在windows使用boost::filesystem::path
category: cpp
tags: boost cpp
---

```cpp
boost::filesystem::path config_file("config");
po::store ( po::parse_config_file<char> ( config_file.c_str(), desc ), vm ); // 这里将会报错,不能正常编译.
```

查看过boost文档后,发现windows,mingw等环境下,value\_type是wchar\_t, 那么path::c\_str()返回的类型是const value\_type*也就是 const wchat\_t*,
没有办法使用program\_options.

继续看文档,发现有一个path::string(), 将代码改成如下

```cpp
boost::filesystem::path config_file("config");
po::store ( po::parse_config_file<char> ( config_file.string().c_str(), desc ), vm ); // OK now.
```

这时候可以正常工作了.
最开始是因为觉得这样子写太麻烦, 就直接用了c\_str(). 没想到在linux上ok, 到windows上却出问题.
