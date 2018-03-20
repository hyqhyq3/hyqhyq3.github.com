---
layout: post
title: Primary script unknown
category: 
tags: SELINUX nignx
---

在配置nginx的时候，出现 Primary script unknown。在检查SCRIPT\_FILENAME等配置后，还是没能解决问题。看到网上说把root设置成/var/www就没问题，猜测可能是SELINUX引起的，用setenforce 0后，nginx正常了。
