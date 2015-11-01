---
layout: post
title: Boost.log链接问题
category: 技术
tags: Boost Boost.log
---

Boost.log是我用过的最难链接的库！！！

使用的时候，有以下几点需要注意：

1.分清是多线程库还是单线程库，`BOOST_LOG_NO_THREADS`可以禁用多线程支持
2.分清是静态链接还是动态链接，如果需要动态链接Boost.log，需要定义宏`BOOST_LOG_DYN_LINK`或者`BOOST_ALL_DYN_LINK`
3.Linux环境下静态链接时，libboost_log_setup.a需要在libboost_log.a前面，libboost_regex.a需要在他们后面

