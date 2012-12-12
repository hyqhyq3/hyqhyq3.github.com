---
layout: post
title: 关闭linux的ICMP Redirect
category: 经验
tags: linux
---

前些天讲公司的linux服务器当成路由器来使用,用于监控流量.但是使用过程中发现有些流量没有完全通过它,而是直接发往路由器了.

经过tcpdump抓包查看,linux往客户端发送了icmp redirect,然后客户端就找到了正确的路由的ip.

	0:80:c8:f8:5c:71 0:80:c8:f8:4a:51 102: 192.168.99.254 > 192.168.99.35: icmp: redirect 192.168.98.82 to host 192.168.99.1 [tos 0xc0] 

这样就没法进行监控和防火墙设置了,没有办法,在网上搜了一阵子,有说修改sysctl.conf

	#sysctl.conf
	net.ipv4.conf.all.send_redirects = 0

但是还是不起作用,最后找到[这篇文章](http://unix.stackexchange.com/questions/57941/linux-always-send-icmp-redirect)

	echo 0 | tee /proc/sys/net/ipv4/conf/*/send_redirects

问题成功解决