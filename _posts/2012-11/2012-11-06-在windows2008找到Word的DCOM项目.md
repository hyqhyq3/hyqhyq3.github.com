---

layout: post
title: 在Windows 2008 r2 中找到Word的组件服务
category: 技术
tags: Windows DCOM

---
因为网页服务器程序需要通过DCOM访问Word和PowerPoint实现文档转换,但是在Windows2008中的组件服务里面默认找不到Office的组件.

经过一些搜索,找到了解决的办法:

1. 运行mmc -32
2. 在弹出的新窗口中,选择**文件**=>**添加/删除管理单元**
3. 然后从左边选择**组件服务**,添加到右边
4. 点击确定,返回到主窗口后,在**控制台根节点**=>**计算机**=>**我的电脑**=>**DCOM配置**里面,就可以找到**Microsoft PowerPoint 幻灯片**和**Microsoft Word 2003-2007**的项目了