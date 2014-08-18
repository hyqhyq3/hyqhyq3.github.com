---
layout: post
title: cocos2d-x的GL Calls优化
category: 技术
tags: cocos2d-x
---

最近一次在手机上运行项目的时候，发现卡得不行。本以为是iOS上不支持luajit，lua执行效率太低，准备用C++改写部分逻辑。在同事的提醒下，发现GL calls初始200+，峰值900+。据说别人游戏优化得好，GL Calls才30+，不管是不是这个问题，先解决了再说。

cocos2d-x 3.X以后的版本，是不需要使用SpriteBatchNode来提升渲染效率，直接往场景上加Sprite就行。但是自动BatchNode也有限制：*使用同一个Texture，并且层次上连续的Sprite才可以自动批处理*。也就是说SpriteA, SpriteB如果他们使用同一个Texture，并且他们之间没有其他Texture的Sprite，那么他们可以在同一个GL call里面绘制出来。
如果SpriteA, SpriteB之间，有一个SpriteC，使用另一个Texture，那么GL calls将会变成3。

优化骨骼动画
---
看看项目里面的情况，由于使用的骨骼动画，每个骨骼动画都对应一个Json文件和一个SpriteSheet。在场景上，各个骨骼动画互相堆叠，导致GL Calls数量非常多。看了一下Json文件，发现里面有个config_png_path和config_file_path，表示与这个Json关联的SpriteSheet。

为了让不同的骨骼动画在同一个GL call中绘制，需要将多个骨骼动画的碎图拼成一个大图，并且让骨骼动画在加载时，不再自动加载SpriteSheet。

##方法：
把骨骼动画源文件的Resources目录里面的碎图全部用TexturePacker打包，输出一个SpriteSheet，然后再把CocosStudio导出的骨骼动画的SpriteSheet删掉（plist和png文件），修改Json文件，将config_png_path和config_file_path改为空数组（这个不改也行，不过会有警告）

优化Atlas
---
优化完骨骼动画后，发现出伤害数字的时候，GL Calls也特别高（游戏里面创建Atlas特别多）。由于不清楚Atlas的内部实现，也不想去改动它的代码，于是自己写了个lua类来实现简单的Atlas。

##方法
将Atlas的png切成碎图，可以使用ImageMagick批量操作，`convert number.png -crop 18x22 number-%02d.png`，然后分解字符串，然后根据对应的字符，加载Sprite，设置位置

优化血条
---
血条其实就是用的一个ProgressTimer，这个ProgressTimer同样存在GL calls的问题, 和Atlas一样，只能自己根据需求撸一个出来

##方法
1. 用2个Sprite，一个是血条的slot，一个是血条的bar，slot和bar的锚点都写成(0, 0.5), slot的位置为(0, 0)，bar的位置为((slot.width - bar.width / 2), 0)
2. 在设置进度的时候，改变bar的scaleX


结果
---
最后将GL calls优化到80左右，感觉上优化得差不多了（有部分UI不好优化）