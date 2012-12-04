---
layout: post
title: go中使用io.ReadFull()
category: 经验
tags: go
---

在写Server的过程中碰到一个问题,我写了如下代码

{%highlight go %}
p := make([]byte, 100)
n, err := r.Read(p)
if n < 100 || err != nil {
	panic(err)
}
{%endhighlight%}

然后执行的时候发现问题了,由于网络上的数据并不一定是一次性到达的,导致n小于100,但是err却是nil

想了一下,把代码改成如下形式
{%highlight go %}
p := make([]byte, 100)
t := 100
for t > 0{
	n, err := r.Read(p)
	t -= n
	if err != nil {
		panic(err)
	}
}
{%endhighlight%}
可是还是觉得有点啰嗦,翻了翻手册,找到了io.ReadAll(r io.Reader, p \[\]byte)

代码如下
{%highlight go %}
p := make([]byte, 100)
n,err := io.ReadAll(r, p) //在遇到EOF或者出错时,n可能小于缓冲区长度
log.Pringf("read %d bytes.", n) 
if err != nil {
	panic(err)
}
{%endhighlight%}