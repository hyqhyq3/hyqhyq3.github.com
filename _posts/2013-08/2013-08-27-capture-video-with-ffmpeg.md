---
layout: post
title: capture video with ffmpeg
category: 技术
tags: FFmpeg
---

在linux下，ffmpeg通过v4l2来抓取摄像头数据（gentoo需要将use里面的v4l2加上）。

如果使用命令行，可以通过`ffmpeg -f v4l2 -i /dev/video0 -vcodec libx264 a.mp4`来抓取数据并编码。

但是命令行局限性太多，特别是如果想将摄像头取到的数据，作为实时流发布，ffmpeg几乎废掉了。

于是我需要通过编程的方式来抓取摄像头数据。

```cpp
// 以下宏是为了避免UINT64_C错误的出现
#ifdef __cplusplus
 #define __STDC_CONSTANT_MACROS
 #ifdef _STDINT_H
  #undef _STDINT_H
 #endif
 #include <stdint.h>
#endif
extern "C" {
#include <libavdevice/avdevice.h>
#include <libavformat/avformat.h>
#include <libavcodec/avcodec.h>
};
#include <iostream>
#include <assert.h>

int main(int argc, char **argv) {
	av_register_all();
	avdevice_register_all();
	
	AVFormatContext *formatCtx = NULL;
	
	AVInputFormat *inputFmt = av_find_input_format("v4l2");
	if(inputFmt == NULL)
	{
		std::cerr << "cannot find input for v4l2, recompile ffmpeg with v4l2" << std::endl;
	}
	if(avformat_open_input(&formatCtx, "/dev/video0", inputFmt, 0) < 0) 
	{
		std::cerr << "cannot open input" << std::endl;
		return -1;
	}
	
	AVPacket pkt;
	if(av_read_frame(formatCtx, &pkt) < 0)
	{
		std::cerr << "cannot read frame" << std::endl;
		return -1;
	}
	
	std::cout << "packet size:" << pkt.size << std::endl;
	
	avformat_close_input(&formatCtx);
}

```