---
layout: post
title: 内核缺少FUTEX配置导致sshd占用过多cpu
category: 
tags: 
---
这几天用ssh连接Gentoo虚拟机时，cpu风扇总是转个不停。用top一看，sshd进程占用了400%的CPU。

使用htop+strace查看了一下系统调用的日志，发现是futex被频繁调用，结果是function not implement，想想应该是上次编译内核时，没有选中这一项。

重新配置编译内核后，问题解决。
