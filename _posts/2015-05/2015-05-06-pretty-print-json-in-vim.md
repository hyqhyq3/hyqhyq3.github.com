---
layout: post
title: 使用vim格式化json
category: 
tags: vim
---

用vim格式化json很简单
```
:%!python -m json.tool
```
意思是对所有行使用命令进行过滤，python -m json.tool就是干活的命令.
