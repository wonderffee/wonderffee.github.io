---
layout: post
title: "iOS实现UIImageView透明区域点击事件穿透"
date: 2013-07-10 22:16
comments: true
categories: 
---

##问题
最近要在iPad上实现一个很独特的功能，简单描述一下就是要显示一个带有半透明背景的弹出界面，在其上加一个不规则形状的图片，手指点击这个弹出界面的半透明区域就退出这个弹出界面。

问题是UED/美工不会提供纯粹的不规则形状切图，实际只能给出的是以不规则形状加透明区域的矩形切图，这就带来另外一个要求：点击矩形切图的透明区域也要退出弹出界面。这就有点难办了，透明区域也是不规则形状的，该怎么判断出手指点击的点就是透明区域呢？

##思路
一般在iOS的控件中，要不就是完全允许用户点击，要不就是禁止用户交互，这是可以通过设置控件的userInteractionEnabled属性来修改。如果添加的图片不是不规则形状的，而是矩形，这问题就简单多了，只需要将矩形图片对应的UIImageView的userInteractionEnabled设为YES，对半透明背景View（或者直接设置为一个按钮）设置点击事件处理，就可以点击实现半透明背景退出弹出界面。

现在的情况是这个矩形图片一分为二，一部分为实体的不规则形状图片，一部分为不规则形状的透明区域。很显然，问题的解决思路是：让手指能“穿透”这个不规则透明区域去点击背后的半透明背景，而不透明部分就不“穿透”。

前面说的userInteractionEnabled属性只是简单地一刀切设置控件是否允许用户操作（即可以响应手指触摸事件），更加灵活的设置方法是使用UIView的hitTest:withEvent:与pointInside:withEvent:。简单介绍下，iOS中的pointInside:withEvent:方法是用来判断当前的点击或者触摸事件的点是否在当前的view中，它被hitTest:withEvent:调用，通过对每个子视图调用pointInside:withEvent:决定最终哪个视图来响应此事件。如果一个子视图的pointInside:withEvent:返回NO，说明这个子视图不会响应点击事件，然后就去寻找更深层的子视图来找到最终响应触摸事件；返回YES就说明子视图能响应点击事件（但不一定是子视图本身响应，若子视图还有子视图的话，还会继续循环去找最终响应事件的子子视图）。

于是，本文的问题就可以这样转化：创建一个UIImageView的子类，重写pointInside:withEvent:方法，让矩形图片的透明区域的pointInside:withEvent:返回NO，而非透明区域的pointInside:withEvent:返回YES，如果能达到这个要求，透明区域点击事件穿透就能够实现。

现在的关键问题是怎么识别出这个透明区域。   
iOS中通常用的图片是PNG图片，这种图片有alpha通道，如果能获取PNG图片每个像素的alpha值，就不难判断出手指点击的图片区域是不是透明的。    

关键代码如下：   

{% codeblock Here's Code lang:objc %}
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    //Using code from http://stackoverflow.com/questions/1042830/retrieving-a-pixel-alpha-value-for-a-uiimage

    unsigned char pixel[1] = {0};
    CGContextRef context = CGBitmapContextCreate(pixel,
                                                 1, 1, 8, 1, NULL,
                                                 kCGImageAlphaOnly);
    UIGraphicsPushContext(context);
    [self.image drawAtPoint:CGPointMake(-point.x, -point.y)];
    UIGraphicsPopContext();
    CGContextRelease(context);
    CGFloat alpha = pixel[0]/255.0f;
    BOOL transparent = alpha < 0.01f;

    return !transparent;
}
{% endcodeblock %}

解释：    
这段代码是通过CGBitmapContextCreate方法创建只包含alpha通道的图形上下文（真不知道context怎么翻译为最好），这个图形上下文的大小为1x1，也就是实际上只放得下一个像素，将矩形图片手指触摸点point绘制到这个图形上下文中，那么pixel数组中唯一元素的值就是手指触摸点那一个像素的alpha值，做归一化为与0.01比较，如果小于0.01就表明手指触摸点是透明的，这时候返回NO就能够实现穿透效果，相反大于0.01就不会穿透。

注意到代码中用到的坐标为(-point.x, -point.y)，为什么会是负数呢？这是因为如果context的区域大小与image一致的话，[image drawAtPoint:]就会将image全部绘制在context中，而实际上context只放得下一个像素，为了保证point点能刚好绘制在这个context上，就必须设置绘制的起始坐标为(-point.x, -point.y)。

代码中的UIGraphicsPushContext容易误导人，看名字以为是将参数中指定的context push入栈，但是参数中的context明明就是刚创建的啊？其实它是将旧的context（默认的context）入栈，再切换到新的context（也就是参数中指定的）绘制，执行UIGraphicsPopContext后就会切换回旧的context，而在新的context上绘制的内容完全不影响旧context（默认context）。这与CGContextSaveGState和CGContextRestoreGState是有本质区别的。


附CGBitmapContextCreate函数参数详解:   
原型：
{% codeblock lang:c %}
CGContextRef CGBitmapContextCreate (    
   void *data,   
   size_t width,    
   size_t height,   
   size_t bitsPerComponent,    
   size_t bytesPerRow,    
   CGColorSpaceRef colorspace,    
   CGBitmapInfo bitmapInfo    
);   
{% endcodeblock %}

参数：    
data                                    指向要渲染的绘制内存的地址。这个内存块的大小至少是（bytesPerRow*height）个字节   
width                                  bitmap的宽度,单位为像素   
height                                bitmap的高度,单位为像素    
bitsPerComponent        内存中像素的每个组件的位数.例如，对于32位像素格式和RGB 颜色空间，你应该将这个值设为8.    
bytesPerRow                  bitmap的每一行在内存所占的比特数    
colorspace                      bitmap上下文使用的颜色空间。    
bitmapInfo                       指定bitmap是否包含alpha通道，像素中alpha通道的相对位置，像素组件是整形还是浮点型等信息的字符串。   

描述：   
当你调用这个函数的时候，Quartz创建一个位图绘制环境，也就是位图上下文。当你向上下文中绘制信息时，Quartz把你要绘制的信息作为位图数据绘制到指定的内存块。一个新的位图上下文的像素格式由三个参数决定：每个组件的位数，颜色空间，alpha选项。alpha值决定了绘制像素的透明性。 


###参考资料：
*   [CGBitmapContextCreate函数参数详解](http://blog.csdn.net/wangyuchun_799/article/details/7804809)
*   [UiView事件传递相关函数：pointInside:withEvent:](http://blog.sina.com.cn/s/blog_489ab04e01010utb.html)
*   [iOS中管理图形上下文](http://wiki.eoe.cn/page/iOS_pptl_artile_28218.html)
*   [Detect touches only on non-transparent pixels of UIImageView, efficiently](http://stackoverflow.com/questions/13291919/detect-touches-only-on-non-transparent-pixels-of-uiimageview-efficiently)