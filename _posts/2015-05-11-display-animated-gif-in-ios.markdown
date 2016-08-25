---
layout: post
title: Display animated GIF in iOS
date: 2015-05-11 15:46
tags: 
---

众所周知，iOS不支持播放GIF格式的动图。问题的原因有几种说法：一种说法是因为最初MacOS不支持Flash，而GIF恰巧是Flash的一种导出格式，所以iOS也就不支持GIF了；另外一种说法是因GIF格式太老，近几年出现的HTML5完全可以取代Flash以及GIF，所以iOS设计之初就不支持GIF。其实真正的原因只有设计iOS系统的人知道。暂且不管这其中的原因，先来看看如何支持GIF动图：

### GIF格式

先从[`GIF`](http://zh.wikipedia.org/wiki/GIF)格式说起，GIF是一种位图格式，与[`矢量图`](http://en.wikipedia.org/wiki/Vector_graphics)相反，它是一种压缩格式，因其体积小而成像相对清晰，广泛用于网络传输。从GIF格式的结构来看，一张完整的GIF图片包含了若干帧，你可以将其视为与zip格式类似的方式将其压缩到一个文件中，当显示时再将其一帧一帧的展示出来，可以在Mac OS X中打开一张GIF图看一下多帧的效果，如下截图：

第1帧：

![](/assets/images/2015/05/gif_frame_1.png)

第45帧：

![](/assets/images/2015/05/gif_frame_2.png)

这样就比较清晰了，其实支持GIF动图，也就是能够把GIF格式压缩的多帧图按照顺序进行展示，并循环这个过程。

### 相关概念

简单介绍一下GIF的几个概念：

* frame（帧）：一个GIF可以简单认为是多张image组成的动画，一帧就是其中一张图片image；
* frameCount（帧数）：表示GIF有多少帧；
* loopCount（播放次数）：有些GIF播放到一定次数就停止了，如果为0就代表GIF一直循环播放；
* delayTime（延迟时间）：每一帧播放的时间，也就是说这帧显示到delayTime就转到下一帧。

所以GIF播放主要就是把每一帧image解析出来，然后每一帧显示它对应的delaytime，然后再显示下一张，如此循环下去。

### 实现展示GIF动图

iOS上实现GIF播放动图有几种方式：基于ImageIO.framework框架、通过UIWebView实现、使用UIImageView的Animating相关方法

* 基于ImageIO.framework框架

这种方式最直接有效，通过ImageIO的CGImageSourceRef读取GIF图片数据，取得CGImageSourceGetCount即frameCount，通过CGImageSourceCreateImageAtIndex方法遍历每一帧的image，同时通过CGImageProperties的kCGImagePropertyGIFDelayTime累计总的delayTime，最后通过UIImage的animatedImageWithImages:duration得到一个animatedImage，在此引用SDWebImage中的UIImage+GIF这个Category，这个Category是引用自LBGIFImage这个开源项目，代码如下：

```
//
//  UIImage+GIF.m
//  LBGIFImage
//
//  Created by Laurin Brandner on 06.01.12.
//  Copyright (c) 2012 __MyCompanyName__. All rights reserved.
//

#import "UIImage+GIF.h"
#import <ImageIO/ImageIO.h>

@implementation UIImage (GIF)

+ (UIImage *)sd_animatedGIFWithData:(NSData *)data
{
    if (!data)
    {
        return nil;
    }
    
    CGImageSourceRef source = CGImageSourceCreateWithData((__bridge CFDataRef)data, NULL);
    
    size_t count = CGImageSourceGetCount(source);

    UIImage *animatedImage;

    if (count <= 1)
    {
        animatedImage = [[UIImage alloc] initWithData:data];
    }
    else
    {
        NSMutableArray *images = [NSMutableArray array];

        NSTimeInterval duration = 0.0f;

        for (size_t i = 0; i < count; i++)
        {
            CGImageRef image = CGImageSourceCreateImageAtIndex(source, i, NULL);

            NSDictionary *frameProperties = CFBridgingRelease(CGImageSourceCopyPropertiesAtIndex(source, i, NULL));
            duration += [[[frameProperties objectForKey:(NSString*)kCGImagePropertyGIFDictionary] objectForKey:(NSString*)kCGImagePropertyGIFDelayTime] doubleValue];

            [images addObject:[UIImage imageWithCGImage:image scale:[UIScreen mainScreen].scale orientation:UIImageOrientationUp]];

            CGImageRelease(image);
        }

        if (!duration)
        {
            duration = (1.0f/10.0f)*count;
        }

        animatedImage = [UIImage animatedImageWithImages:images duration:duration];
    }

    CFRelease(source);

    return animatedImage;
}
```

* 通过UIWebView实现 
这种方式与在webview上展示其他格式的静态图片一样，代码如下：

```
	NSString *path = [[NSBundle mainBundle] pathForResource:@"run" ofType:@"gif"];
    NSData *gifData = [NSData dataWithContentsOfFile:path];
    UIWebView *webView = [[UIWebView alloc] initWithFrame:CGRectMake(0, 120, 100, 100)];
    [webView loadData:gifData MIMEType:@"image/gif" textEncodingName:nil baseURL:nil];
    [self.view addSubview:webView];
```


* 使用UIImageView的Animating相关方法
此种方式是将GIF预先转换成单帧图，使用单帧图名称初始化一个数组，设置animationDuration，通过startAnimating方法进行展示，代码如下：

```
	UIImageView *gifImageView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 240, 100, 100)];
    NSArray *gifArray = [NSArray arrayWithObjects:[UIImage imageNamed:@"1"],
                         [UIImage imageNamed:@"2"],
                         [UIImage imageNamed:@"3"],
                         [UIImage imageNamed:@"4"],
                         [UIImage imageNamed:@"5"],
                         [UIImage imageNamed:@"6"],
                         [UIImage imageNamed:@"7"],
                         [UIImage imageNamed:@"8"],
                         [UIImage imageNamed:@"9"],
                         [UIImage imageNamed:@"10"],nil];
    gifImageView.animationImages = gifArray; 
    gifImageView.animationDuration = 5; 
    gifImageView.animationRepeatCount = 999;
    [gifImageView startAnimating];
    [self.view addSubview:gifImageView];
```
这种方式有个限制条件是事先需要将GIF图转为单帧图，所以此方式适合做一次性处理，如果放到动态加载图片的程序中，还需要写个将GIF图解码为单帧图的函数。

综合上面三种方式可知，第一种方式实际是使用了CoreGraphics的相关API，将GIF数据解析为单帧图再将其转换为animatedImage，整个流程实际是进行数据解析和转换的过程；第二种是通过UIWebView以保持原有帧速的方式实现；第三种方式，实际是使用了定时器来控制图片的播放从而模拟成动图。第一种和第三种方式在模拟的帧速不准确时，可能与原图效果不符。而第一种方式使用了CoreGraphics相对更底层的API，这种方式可能性能更高一些，也是目前使用最多的方式。

一些开源GIF项目也都是使用第一种方式实现的，GitHub上有几个star较多的项目，对几个比较受欢迎的项目简单做了一下对比：

* [`FLAnimatedImage`](https://github.com/Flipboard/FLAnimatedImage) 

Flipboard团队出品，一直在维护。特点是兼容性好，如果项目中同时使用其他第三方框架建议使用，不用担心冲突问题，因为只需要在需要支持GIF图的代码中使用此框架的Image和ImageView相关方法即可，不会影响其他代码。

* [`AnimatedGIFImageSerialization`](https://github.com/mattt/AnimatedGIFImageSerialization)

出自著名的AFNetworking作者mattt大神之手，与AFN一样，因其重写了UIImage相关方法，使用方便，只需要将头文件引入即可使UIImage支持GIF，还可以将数据序列化为NSData，但缺点是引入此框架后会将整个项目的UIImage方法替换为提重写后的方法，有可能与其他代码产生冲突。

* [`SDWebImage 的 UIImage+GIF`](https://github.com/rs/SDWebImage)

特点是简单易用，在需要支持GIF的位置加上一个方法即可，不必担心冲突。

参考：

[`How Flipboard plays animated GIFs on iOS`](http://engineering.flipboard.com/2014/05/animated-gif/)

[`CGImageSource对图像数据读取任务的抽象`](http://www.tanhao.me/pieces/1019.html/)

[`GIF图片的预览以及生成`](http://bytelee.me/blog/2013/05/16/ios-giftu-pian-de-yu-lan-yi-ji-sheng-cheng/)

[`iOS由ImageIO.framework实现gif的系统解码`](http://www.tuicool.com/articles/RZJRrmb)

[`我对几个受欢迎开源项目的对比_Github`](https://github.com/yuxiaopeng/GIF)

【完】

