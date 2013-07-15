---
layout: post
title: 在Gentoo上设置usbnet
category: 
tags: Gentoo
---

Linux的无线网卡是个大问题，在无线网卡不好用的时候，我用手机作为无线网卡使用。

手机连上wifi后，用usb和电脑连接，选择usb共享，这时候dhcpcd usb0一下，就可以上网了。

不过在Gentoo上，得在编译内核时，开启如下模块

···
Device Drivers ---> 
  [*] Network device support ---> 
    USB Network Adapters ---> 
      [*] Multi-purpose USB Networking Framework 
        <*> CDC Ethernet support 
        <*> CDC EEM support 
        <*> Simple USB Network Links (CDC Ethernet subset) 
          [*] Embedded ARM Linux links 
  [*] USB Support ---> 
    <*> USB Modem (CDC ACM) support 
    <*> USB Wireless Device Management support 
···