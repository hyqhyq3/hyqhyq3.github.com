---
layout: post
title: 使用FFmpeg编写音乐播放器
category: 
tags: FFmpeg
---

本文介绍了用ffmpeg编写一个简单的音乐播放器, 适合初学者入门一下. 

本文用[FFmpeg](http://ffmpeg.org)进行音频解码,用[PortAudio](http://www.portaudio.com)播放声音.

{% highlight c++ %}
#include <iostream>
#include <assert.h>
#include <gst/gst.h>

int main(int argc, char* argv[]) {
	gst_init(&argc, &argv);
	GstElement *source = gst_element_factory_make("pulsesrc", "source");
	if(! source)
	{
		std::cout << "cannot find pulse" << std::endl;
	}
	GstElement *encoder = gst_element_factory_make("speexenc", "encoder");
	if(! encoder)
	{
		std::cout << "cannot open encoder" << std::endl;
	}
}

{% endhighlight %}
