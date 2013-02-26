---
layout: post
title: 通过HTTPS代理科学上网
category: 科学上网
tags: Proxy
---

#前言
考虑到当今严峻的上网形势,使用一种小众的,高强度加密的科学上网方法迫在眉睫.

这里介绍的是使用squid的ssl功能来对传输过程进行加密,以避免传输过程中被干扰.

#特点
1. 安全:客户端与服务器之间,通过SSL传递数据.客户端和服务器端都严格控制证书的有效性,防止中间人攻击
2. 简便:个人觉得这已经是一种很简单的,成本很低的配置方式了

#要求
1. 有个墙外的VPS
2. 有个懂得敲命令的linux玩家

#服务器配置
1. 安装squid, 一般来说直接用apt或者yum等包管理工具就能安装. 安装完成后,先看看有没有ssl(--enable-ssl)支持,如果没有ssl支持,可能需要手动编译.
	
	[root@li384-23 ~]# squid -v
	Squid Cache: Version 3.2.5
	configure options:  '--build=x86_64-unknown-linux-gnu' '--host=x86_64-unknown-linux-gnu' '--target=x86_64-redhat-linux-gnu' '--program-prefix=' '--prefix=/usr' '--exec-prefix=/usr' '--bindir=/usr/bin' '--sbindir=/usr/sbin' '--sysconfdir=/etc' '--datadir=/usr/share' '--includedir=/usr/include' '--libdir=/usr/lib64' '--libexecdir=/usr/libexec' '--sharedstatedir=/var/lib' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--exec_prefix=/usr' '--libexecdir=/usr/lib64/squid' '--localstatedir=/var' '--datadir=/usr/share/squid' '--sysconfdir=/etc/squid' '--with-logdir=$(localstatedir)/log/squid' '--with-pidfile=$(localstatedir)/run/squid.pid' '--disable-dependency-tracking' '--enable-arp-acl' '--enable-follow-x-forwarded-for' '--enable-auth' '--enable-basic-auth-helpers=LDAP,MSNT,NCSA,PAM,SMB,YP,getpwnam,multi-domain-NTLM,SASL,DB,POP3,squid_radius_auth' '--enable-ntlm-auth-helpers=smb_lm,no_check,fakeauth' '--enable-digest-auth-helpers=password,ldap,eDirectory' '--enable-negotiate-auth-helpers=squid_kerb_auth' '--enable-external-acl-helpers=ip_user,ldap_group,session,unix_group,wbinfo_group' '--enable-cache-digests' '--enable-cachemgr-hostname=localhost' '--enable-delay-pools' '--enable-epoll' '--enable-icap-client' '--enable-ident-lookups' '--enable-linux-netfilter' '--enable-referer-log' '--enable-removal-policies=heap,lru' '--enable-snmp' '--enable-ssl' '--enable-storeio=aufs,diskd,ufs' '--enable-useragent-log' '--enable-wccpv2' '--enable-esi' '--with-aio' '--with-default-user=squid' '--with-filedescriptors=16384' '--with-dl' '--with-openssl' '--with-pthreads' 'build_alias=x86_64-unknown-linux-gnu' 'host_alias=x86_64-unknown-linux-gnu' 'target_alias=x86_64-redhat-linux-gnu' 'CFLAGS=-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -fpie' 'CXXFLAGS=-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -fpie' 'PKG_CONFIG_PATH=/usr/lib64/pkgconfig:/usr/share/pkgconfig'
	
2. 生成ca和证书,这个网上也有相关的文章,这里提供一种比较简便的方法. openvpn提供了一个easy-rsa的脚本,可以很方便地创建ssl证书,我这里就用它来创建证书

	$ yum install openvpn
	$ cp /usr/share/openvpn/easy-rsa/2.0 ~/easy-rsa
	$ cd ~/easy-rsa
	$ source vars # 如果这一步提示缺少openssl.cnf,那么请查看下当前目录下有没有openssl-x.x.x.cnf的文件,将他重命名一下
	$ ./clean-all
	$ ./build-dh
	$ ./build-ca
	$ ./build-key-server example.com # 这里替换成你的域名,最好是用你服务器真实的域名
	$ ./build-key user1 #这里替换成你的用户名,实际上问题不太大的
	
3. 将easy-rsa/keys里面的ca.key ca.crt好好保存着,用于签发证书,将ca.crt example.com.crt example.com.key放到服务器的/etc/squid/里面,将ca.crt user1.crt user1.key放在客户端.

4. 修改squid的配置文件
	
	......
	
	http_access allow all #加上这句,并注释掉下面那句deny all
	# http_access deny all
	
	# http_port 3128 #这行也注释掉,我们不需要http proxy
	https_port 5678 cert=/etc/squid/example.com.crt key=/etc/squid/example.com.key cafile=/etc/squid/ca.crt
	
修改完配置后,重启squid服务器

5. 客户端配置

客户端可以继续使用squid,不过我觉得不用使用那么重量级的东西了,于是我选择了socat.

	socat tcp-listen:5678,reuseaddr,fork openssl:example.com:5678,cert=/path/to/user1.crt,key=/path/to/user1.key,cafile=/path/to/ca.crt
	
这样就可以运行一个http://127.0.0.1:5678/的代理, 给firefox安装autoproxy,并设置127.0.0.1 5678 http 成默认代理,就ok了.

每次启动一下肯定很麻烦,于是我写了一个systemd的unit文件

	# /usr/lib/systemd/system/proxy.service
	[Unit]
	Description=OpenSSH Daemon
	Wanted=network.target

	[Service]
	EnvironmentFile=/etc/proxy.conf
	ExecStart=/usr/bin/socat tcp-listen:${listen-port},reuseaddr,fork openssl:${server},cafile=${cafile},cert=${cert},key=${key}
	Restart=always

	[Install]
	WantedBy=multi-user.target
	
还有配置文件

	# /etc/proxy.conf
	listen-port=5678
	server=example.com:5678
	cafile=/path/to/ca.crt
	cert=/path/to/user1.crt
	key=/path/to/user1.key