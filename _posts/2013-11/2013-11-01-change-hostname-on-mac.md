---
layout: post
title: 在OSX下更改主机名
category: 技巧
tags: OSX Mac
---

安装黑苹果后，发现终端显示的主机名为localhost。

在[系统设置]->[共享]->[电脑名称]里面修改了，也没有效果。

在网上搜了一下，发现可以用以下命令来修改主机名

```sh
sudo scutil --set HostName hyq-mac
```

执行完成后重新开启终端，就可以看到效果
