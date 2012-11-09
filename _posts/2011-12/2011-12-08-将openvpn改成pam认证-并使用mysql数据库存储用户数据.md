---
layout: post
title: 将OpenVPN改成PAM认证-并使用mysql数据库存储用户数据
category: 服务器
tags: linux openvpn pam mysql
---

修改/etc/openvpn/server.conf，加上以下几句
	
	plugin /usr/lib/openvpn/openvpn-auth-pam.so openvpn    
	client-cert-not-required
	username-as-common-name

然后创建/etc/pam.d/openvpn这个文件，内容如下

	auth required pam_mysql.so  user=vpn passwd=vpn host=localhost db=openvpn table=user usercolumn=username passwdcolumn=password crypt=2    
	account required pam_mysql.so   user=vpn passwd=vpn host=localhost db=openvpn table=user usercolumn=username passwdcolumn=password crypt=2

然后在mysql里面新建数据库openvpn，
新建用户vpn,密码vpn，
新建一个user表，包含username和password两个字段，
添加一个用户 

	insert into user (username,password) values ('test',password('test'))

最后重启OpenVPN

	service openvpn restart
