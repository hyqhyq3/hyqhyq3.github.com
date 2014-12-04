---
layout: post
title: win64平台下context的bug
category: 技术
tags: win64 boost asio context coroutine
---

在使用boost.asio的stackful协程时，windows平台在调用系统api时报错。

仔细检查代码后，发现只要在spawn创建的协程里面使用任何socket api，都有可能让程序直接跪掉。使用win32的boost编译却没事。

在群里面说明问题后，经讨论发现，是boost.context的实现问题，据说作者是靠猜来实现的……

解决办法暂时未知，只能改道用stackless协程，或者放弃win64的支持。	

