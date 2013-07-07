---
layout: post
title: avformat_open_input是一个坑
category: 技术
tags: ffmpeg
---

avformat_open_input是ffmpeg的常用的打开文件函数

    /**
     * Open an input stream and read the header. The codecs are not opened.
     * The stream must be closed with av_close_input_file().
     *
     * @param ps Pointer to user-supplied AVFormatContext (allocated by avformat_alloc_context).
     *           May be a pointer to NULL, in which case an AVFormatContext is allocated by this
     *           function and written into ps.
     *           Note that a user-supplied AVFormatContext will be freed on failure.
     * @param filename Name of the stream to open.
     * @param fmt If non-NULL, this parameter forces a specific input format.
     *            Otherwise the format is autodetected.
     * @param options  A dictionary filled with AVFormatContext and demuxer-private options.
     *                 On return this parameter will be destroyed and replaced with a dict containing
     *                 options that were not found. May be NULL.
     *
     * @return 0 on success, a negative AVERROR on failure.
     *
     * @note If you want to use custom IO, preallocate the format context and set its pb field.
     */
    int avformat_open_input(AVFormatContext **ps, const char *filename, AVInputFormat *fmt, AVDictionary **options);

可以看到第一个参数,既可以是一个NULL的指针,又可以是由avformat_alloc_context()创建的AVFormatContext对象.

所以在使用这个函数的时候,要么保证ps指向一个已分配的内存.要么为NULL

{% highlight c %}
int main() {
    // 这种写法是错误的
    // AVFormatContext *formatContext; 
    
    // 以下两种写法都可以
    // AVFormatContext *formatContext = NULL;
    AVFormatContext *formatContext = avformat_alloc_context();

    AVFormatContext *formatContext; 
    avformat_open_input(&formatContext, "foo.mp3", NULL, NULL);
    return 0;
}
{% endhighlight %}