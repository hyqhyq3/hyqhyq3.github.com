---
layout: post
title: AudioRecord未释放导致初始化错误
category: Android
tags: 
---

昨天碰到一个AndroidRecord不能初始化的问题,折腾了整整一天,还是没有结果. 今天周末,还是在想办法弄清楚这个问题.

在Google上搜索AudioRecord 20后,看到StackOverflow上类似的问题,仔细全部阅读后,发现有人说"Had the same error, till I restarted the device.", 也就是说重启就OK.

这么说来,不是我程序的问题. 开启手机自带的录音机程序,果然也没法用了. 说不定就是因为我的程序运行成功过,然后没有释放麦克风,导致了这个问题.

