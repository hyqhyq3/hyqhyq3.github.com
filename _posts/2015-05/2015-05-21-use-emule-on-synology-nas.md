---
layout: post
title: 在群晖NAS上使用电驴
category: 
tags: NAS emule
---

用群晖自带的Download Station，开启emule下载后，总是不能连接到ed2k服务器和kad网络。直接添加ed2k服务器好像也没用。

费了一些工夫，在网上找到了解决方案。

先关闭emule下载，然后通过ssh或者telnet登陆上nas，执行如下命令

```
	cd /usr/syno/etc/packages/DownloadStation/amule/
	rm server.met nodes.dat
	wget http://upd.emule-security.org/server.met
	wget http://upd.emule-security.org/nodes.dat
```

然后重新开启emule下载，就可以连接到ed2k服务器和kad网络了。