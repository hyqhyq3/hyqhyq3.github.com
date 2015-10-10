---
layout: post
title: 编程实现修改路由表
category: 技术
tags: unix
---

通过程序修改路由表，可以使用`PF_ROUTE`类型的socket（这只是方法之一）。

通过`socket(PF_ROUTE, SOCK_RAW, 0)`创建一个socket，然后向这个socket写类型为`rtmsg_hdr`和`struct sockaddr`的数据，就可以操作路由表。

以下是详细代码：


```
#include <net/route.h>
#include <arpa/inet.h>
#define BUFLEN (sizeof(struct rt_msghdr) + 512)
#define SEQ 9999
int add_route_for_host(const char* target, const char* route) {
	pid_t pid;
	int sockfd = socket(AF_ROUTE, SOCK_RAW, 0);
	char* buf = malloc(BUFLEN);
	struct rt_msghdr *rtm = (struct rt_msghdr*)buf;
	rtm->rtm_msglen = sizeof(struct rt_msghdr) + sizeof(struct sockaddr_in) * 2;
	rtm->rtm_version = RTM_VERSION;
	rtm->rtm_type = RTM_ADD;
	rtm->rtm_flags = RTF_HOST;
	rtm->rtm_addrs = RTA_DST | RTA_GATEWAY;
	rtm->rtm_pid = pid = getpid();
	rtm->rtm_seq = SEQ;

	struct sockaddr_in *sin = (struct sockaddr_in*)(rtm + 1);
	sin->sin_family = AF_INET;
	sin->sin_len = sizeof(struct sockaddr_in);
	inet_pton(AF_INET, target, &sin->sin_addr);

	sin = (struct sockaddr_in*)(sin + 1);
	sin->sin_family = AF_INET;
	sin->sin_len = sizeof(struct sockaddr_in);
	inet_pton(AF_INET, route, &sin->sin_addr);

	write(sockfd, rtm, rtm->rtm_msglen);
	do {
		read(sockfd, rtm, BUFLEN);
	} while(rtm->rtm_type != RTM_ADD || rtm->rtm_seq != SEQ || rtm->rtm_pid != pid);
	return rtm->rtm_errno;
}

int main(int argc, char** argv) 
{
	if(argc == 3) {
		int ret = add_route_for_host(argv[1], argv[2]);
		if(ret != 0) {
			perror("setup route");
		}
	}
}
```

这段代码比较重要的就是rtmsg_hdr后面的sockaddr数据的顺序是有规定的。按照RTA_DST，RTA_GATEWAY，RTA_NETMASK……的顺序。

另外，在读取路由表的时候，返回的数据也按照这个格式。

更多详情可以参考《Unix网络编程》18章 路由套接字
