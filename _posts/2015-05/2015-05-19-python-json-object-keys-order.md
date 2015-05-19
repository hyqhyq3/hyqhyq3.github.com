---
layout: post
title: Python处理json时的key order问题
category: 
tags: python json
---

在用Python处理Cocos的UI文件时，发现python处理过的json文件导致cocostudio崩溃。
刚开始以为是utf8的问题，加上ensure\_ascii=False以后，还是一样的结果。
经手工调整json后，才知道cocostudio对于json的格式有要求（有病）。需要将\_\_type放到object的开头。
但是使用json的sort\_keys并不能解决问题，\_\_type被放到了ZOrder后。

又去网上一阵搜索，找到了一个OrderedDict，在解析json时加上参数object\_pairs\_hook=collections.OrderedDict。


```python
decoder = json.JSONDecoder(object_pairs_hook=collections.OrderedDict)
decoder.decode('{"foo":1,"bar":2}')
```