---
layout: post
title: hosts导致打开samba共享目录超慢的问题
category: 技术
tags: 
---

最近感觉打开nas的共享文件夹超级慢，改了无数次smb.conf，都不能解决这个问题。

今天用smbclient的时候，发现有输出调试日志的选项就想看看有没有什么线索。

```
smbclient -L 192.168.2.117 -U guest -d 9
```

然后发现卡在 
```
Connecting to 192.168.2.117 at port 445
```

到网上搜了下，ubuntu论坛说可能是hosts的原因。  于是查看下/etc/hosts， 想到可能是因为自动更新hosts的脚本没有增加本机IP的问题。

然后加上以下几行
```
192.168.2.117 localhost hyq-ds415
172.27.0.117 localhost hyq-ds415
```

重启samba服务，果然解决了问题。
