---
layout: post
title: 使用FFmpeg编写音乐播放器
category: 
tags: FFmpeg
---

本文介绍了用ffmpeg编写一个简单的音乐播放器, 适合初学者入门一下. 

本文用[FFmpeg](http://ffmpeg.org)进行音频解码,用[PortAudio](http://www.portaudio.com)播放声音.

{% highlight c++ %}
/// main.cpp

// 这里是ffmpeg的头文件, 如果是用c++, 必须加上extern "C", 否则可能导致链接时出错.
extern "C" {
#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
}

// 这里是PortAudio的头文件
#include <portaudio.h>

#include <iostream>


int main(int argc, char* argv[]) 
{
	// 将要打开的音频文件(视频文件也可以支持).
	const char* filename = argc > 1 ? argv[1] : "1.mp3";
	
	// 初始化libavformat,并注册所有的模块
	av_register_all();
	
	// 这里一定要设置成NULL, 或者调用avformat_alloc_context分配内存, 否则可能崩溃.
	AVFormatContext *formatContext = NULL;
	
	// 打开输入文件.
	if( avformat_open_input(&formatContext, filename, NULL, NULL) < 0) {
		std::cerr << "cannot open file" << std::endl;
		return -1;
	}

	// 探测文件里面的音视频流信息.
	if( avformat_find_stream_info(formatContext, NULL) < 0) {
		std::cerr << "cannot find stream info" << std::endl;
		return -1;
	}

	// 输出来看看.
	av_dump_format(formatContext, 0, 0, 0);

	// 找到音频流的索引(如果是视频的话,可能存在多个流).
	int audioIndex;
	if((audioIndex = av_find_best_stream(formatContext, AVMEDIA_TYPE_AUDIO, 0, 0, NULL, 0)) < 0) {
		std::cerr << "cannot find audio stream" << std::endl;
		return -1;
	}

	AVCodecContext *codecContext = formatContext->streams[audioIndex]->codec;
	// 找到解码器.
	AVCodec *codec = avcodec_find_decoder(codecContext->codec_id);
	if(codec == NULL) {
		std::cerr << "cannot find decoder for " << codecContext->codec_name << std::endl;
	}

	// 打开解码器.
	if( avcodec_open2(codecContext, codec, NULL) < 0) {
		std::cerr << "cannot open decoder" << std::endl;
		return -1;
	}

	// AVPacket是解码前的数据, AVFrame是解码后的数据.
	AVPacket packet;
	AVFrame *frame = avcodec_alloc_frame();
	int got;

	// 下面是初始化PortAudio, 用PortAudio的Blocking API比较简单.
	PaStream *stream;
	Pa_Initialize();
	Pa_OpenDefaultStream(&stream, 0, codecContext->channels, 
		paInt16, codecContext->sample_rate, 
		1024, NULL, NULL);
	Pa_StartStream(stream);

	int size = 4092;
	int16_t* buffer = new int16_t[size];
	
	while(true) {
		// 从文件中读取一帧.
		if(av_read_frame(formatContext, &packet) < 0) {
			// 文件读完了.
			break;
		}

		// 解码.
		if( avcodec_decode_audio4(codecContext, frame, &got, &packet) < 0) {
			std::cerr << "cannot decode" << std::endl;
			// 偶尔会出错,一般都可以原谅的...
			// break;
		}

		// 解码出来了一帧
		if(got) {
			// 因为frame->data[0]表示的是左声道LLL....,frame->data[1]表示右声道RRR...
			// 而PortAudio要求的是LRLRLR....这样的数据排布, 所以这里用循环重新将数据复制到buffer中
			for(int i = 0; i < frame->nb_samples; i++) {
				for(int j = 0; j < channels; j ++) {
					buffer[i*channels + j] = reinterpret_cast<int16_t*>(frame->data[j])[i];
				}
			}
			Pa_WriteStream(stream, buffer, frame->nb_samples);
		}

	}
	delete buffer;
	return 0;	
}

{% endhighlight %}
