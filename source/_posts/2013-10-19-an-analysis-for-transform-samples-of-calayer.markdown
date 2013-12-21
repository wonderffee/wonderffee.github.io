---
layout: post
title: "iOS 3D UI——CALayer的transform扩展解析"
date: 2013-10-19 17:44
comments: true
categories: iOS
---
###引言
这篇文章的主要内容来自于CocoaChina论坛上的一篇文章，只不过原文在有些地方介绍得不是很详细，我这里增加了一些解析，也算是自己做笔记，原文和代码均可以在这个链接里找到：[IOS 3D UI --- CALayer的transform扩展](http://www.cocoachina.com/bbs/read.php?tid=117061&page=1)。
<!--more-->
iOS的UI是基于UIView类的，我们能看到的每个UI元素都是UIView或者UIView的子类。View按树形结构组织起来，树根是UIWindow。

###UIView与CALayer
View负责界面的交互和显示，其中显示部分由CALayer来完成。每个UIView包含一个CALayer实例。可以这么认为，UIView本身是不可见的，我们能看到的都是CALayer，UIView只是负责对CALayer进行管理。

UIView的显示设置都是对CALayer属性的封装，但是这层封装掩盖了CALayer提供的3D显示功能。所以我们想让UIView显示3D的效果的话，需要直接操作CALayer。

要操作CALayer对象，首先要在工程中包含QuartzCore.framework，在文件中import <QuartzCore/QuartzCore.h>头文件。QuartzCore.framework中包含了CALayer以及CALayer一些官方子类的定义。
    
通过设置CALayer的transform属性，可以使CALayer产生3D空间内的平移、缩放、旋转等变化。


###绕坐标轴的旋转

原始场景如图：

![test](/images/CALayerTransform/CALayerOrigin.png)

使用image.layer.transform = CATransform3DMakeRotation(M_PI/6, 0, 1, 0); 绕Y轴旋转30度后的效果:

![test](/images/CALayerTransform/CALayerRotation30y.png)

可以发现，绕Y轴旋转只是在X轴上进行了缩放，这是因为，在CALayer的显示系统中，默认的相机使用正交投影，正交投影没有远小近大效果，所以在本例中，只能造成相应轴上的缩放。在这种情况下，无论是绕Y轴旋转30度还是-30度都是同样的效果。

###透视投影
CALayer默认使用正交投影，因此没有远小近大效果，而且没有明确的API可以使用透视投影矩阵。所幸可以通过矩阵连乘自己构造透视投影矩阵。构造透视投影矩阵的代码如下：
```objc
    CATransform3D CATransform3DMakePerspective(CGPoint center, float disZ)
    {
        CATransform3D transToCenter = CATransform3DMakeTranslation(-center.x, -center.y, 0);
        CATransform3D transBack = CATransform3DMakeTranslation(center.x, center.y, 0);
        CATransform3D scale = CATransform3DIdentity;
        scale.m34 = -1.0f/disZ;
        return CATransform3DConcat(CATransform3DConcat(transToCenter, scale), transBack);
    }

    CATransform3D CATransform3DPerspect(CATransform3D t, CGPoint center, float disZ)
    { 
        return CATransform3DConcat(t, CATransform3DMakePerspective(center, disZ));
    }
```

代码中，center指的是相机 的位置，相机的位置是相对于要进行变换的CALayer的来说的，原点是CALayer的anchorPoint在整个CALayer的位置，例如CALayer的大小是(320, 160), anchorPoint值为(0.5, 0.5)，此时anchorPoint在整个CALayer中的位置就是(160, 80)，正中心的位置。传入透视变换的相机位置为(0, 0)，那么相机所在的位置相对于CALayer就是(160, 80)。如果希望相机在左上角，则需要传入(-160, -80)。disZ表示的是相机离z=0平面（也可以理解为屏幕）的距离。

怎么解释这段代码呢？
第一步，layer在CATransform3DPerspect方法中首先进行了t变换，要注意的是这时候进行的t变换是以anchorPoint为中心点，默认情况下是在layer的中心位置。

第二步，在CATransform3DMakePerspective方法先进行了一个(-center.x, -center.y, 0)的平移，然后进行了透视投影，最后又做了一个(center.x, center.y, 0)平移。关键在于这两个反向的平移。
当center为(0,0)也就是相机中心在CALayer的中心位置，与anchorPoint(0.5, 0.5)重合时，平移的距离为0也就是没有做平移，这时候是直接把已经做过t变换的layer通过scale进行透视投影变换的。

当center不在中心位置时，假设在CALayer的左上角，那么center为(-160, -80)。那center就平移到了(0,0)点，等于把相机点又平移回到了与anchorPoint重合的这一点，但由于平移此时这个重合点成矩形图片的左上角了。那么为什么要平移至使相机点与anchorPoint点重合呢？

这里得明确一点，相机点与layer同时在X-Y平面上做相同的偏移时，因为没有改变z值，在相机点看到的立体效果是相同的，只是相对原点的位置变动了而已。在相机点(-160, -80)看到的立体效果，就等效于在相机点(0,0)看到的把layer平移(160, 80)的立体效果.对一个layer来说，只要没有修改anchorPoint，系统所认为的内部相机点的投影是在anchorPoint这个位置，也就是相机点的(0,0)位置。因此要看到layer在相机点(-160, -80)透视投影的效果，只能先作平移变换，让相机点与layer做相同的平移使相机点移到(0,0), 完成透视投影后再平移回去。


带透视效果的绕Y轴旋转，效果如下：  
![test](/images/CALayerTransform/CALayerProject.png)

相应的代码为：
```objc
CATransform3D rotate = CATransform3DMakeRotation(M_PI/6, 0, 1, 0);
image.layer.transform = CATransform3DPerspect(rotate, CGPointMake(0, 0), 200);
```

可以发现在进行逆时针旋转30度时，在中心点左侧的图离相机点比较近，呈现出了比原图大的效果，右侧的图离相机点比较远，呈现出了比原图小的效果。对比原图，图的左边界超出了屏幕，而右边界在屏幕之内，这可以通过下面的这个图来解释：  
![test](/images/CALayerTransform/CALayerProject2.png)

图中PQ为旋转后的图像在X-Z平面上的投影，相机点在O点，从O点看过过，P、Q两点在X轴上的投影为C、D点，C、D两点相对P、Q点在X轴上的原始位置都是靠左的，这也就解释了旋转后的透视投影效果左边界超出了屏幕，右边界在屏幕之内。

上图中A、B两点是O点在无穷远处所看到的P、Q两点在X轴上的投影，也就是我们在第二张图片上看到的效果。这是因为在无穷远处，观察P、Q时已经失去了近大远小的效果，所做的投影是正交投影。

如果把center设为(-160,-80),也就是把相机位置设为矩形图片的左上角，则绕Y轴旋转的透视效果如下：  
![test](/images/CALayerTransform/CALayerProject3.png)



###立方体效果
CATransform3D可以使用CATransform3DConcat函数连接起来以构造更复杂的变换, 通过这些方法，可以组合出更多的效果来。下面是个翻转的动画, 使用四张同样大小的图片围成一个框，让这个框动画旋转, 形成一个立方体旋转的效果。  
![test](/images/CALayerTransform/CALayerCube.png)

相关实现的代码：  
```objc
CATransform3D move = CATransform3DMakeTranslation(0, 0, 160);
CATransform3D back = CATransform3DMakeTranslation(0, 0, -160);

CATransform3D rotate0 = CATransform3DMakeRotation(-angle, 0, 1, 0);
CATransform3D rotate1 = CATransform3DMakeRotation(M_PI_2-angle, 0, 1, 0);
CATransform3D rotate2 = CATransform3DMakeRotation(M_PI_2*2-angle, 0, 1, 0);
CATransform3D rotate3 = CATransform3DMakeRotation(M_PI_2*3-angle, 0, 1, 0);

CATransform3D mat0 = CATransform3DConcat(CATransform3DConcat(move, rotate0), back);
CATransform3D mat1 = CATransform3DConcat(CATransform3DConcat(move, rotate1), back);
CATransform3D mat2 = CATransform3DConcat(CATransform3DConcat(move, rotate2), back);
CATransform3D mat3 = CATransform3DConcat(CATransform3DConcat(move, rotate3), back);

image0.layer.transform = CATransform3DPerspect(mat0, CGPointZero, 500);
image1.layer.transform = CATransform3DPerspect(mat1, CGPointZero, 500);
image2.layer.transform = CATransform3DPerspect(mat2, CGPointZero, 500);
image3.layer.transform = CATransform3DPerspect(mat3, CGPointZero, 500);
```

解析：  
要形成一个立方体旋转的效果，首先需要构造出一个立方体出来，怎么构造呢？在这个例子里构造的立方体是前后左右四个面的，如果把屏幕当做立方体的“前”面，它的“左”、“后”、“右”面我们是看不见，但是这三个面可以通过“前”面旋转一个角度得到的：以立方体的中心点为支点，将“前”面分别顺时针旋转90度、180度、270。因为屏幕宽度为320，这个立方体的中心点应在屏幕中心点后方160px的地方。

现在需要解决的一个问题就是：怎么实现以立方体的中心点为支点的旋转。我们知道，在CALayer中layer的旋转是以anchorPoint为支点的，而这个anchorPoint并没有在z轴上的维度，所以修改anchorPoint是不可能的，怎么办呢？答案还是通过平移实现，虽然不能修改anchorPoint，但我们可以改变图片的位置，将图片往z轴正方向(靠近用户的方向)平移160px的距离，这时候图片与anchorPoint的相对位置，就等同于图片在原始位置与立方体中心的相对位置，它们所进行的旋转效果是相同的，只是在z轴上的绝对距离不同。旋转完成后，再平移回去，即可达到绕立方体的中心点旋转的效果。这也是变换矩阵mat0为什么要先进行z轴正方向160px平移，执行rotate0旋转之后又进行z轴负方向160px平移的缘故。

要实现旋转动画，就需要动态改变这个立方体的绕y轴的角度，在这个例子里就是添加了一个动态变化的angle达到这个目的。另外注意此例的旋转是绕y轴旋转的，根据此前一篇文章的判断方法，此时旋转的正方向应该是z轴正方向顶点指向x轴正方向顶点，从用户眼睛看来就是逆时针。如果angle是递增的，那么-angle就是递减的，因此实际看到的旋转动画会是顺时针的。


在分析这个例子时，自己又突然想到另外一个问题：对一个layer做平移，会修改它的anchorPoint和position吗？很显然，对旋转和绽放必须要有一个固定的支点，感觉上平移不需要支点也行，是不是就会修改anchorPoint呢？答案是否定时，简单做一下测试，就知道layer在做平移时，anchorPoint和position都不会改变，frame会变化，说明frame不仅受anchorPoint和position影响，还受translation影响.

####参考链接
* [IOS 3D UI --- CALayer的transform扩展](http://www.cocoachina.com/bbs/read.php?tid=117061&page=1)
* [iOS的三维透视投影](http://geeklu.com/2012/07/ios-3d-perspective/)
* [学习笔记5（CATransform3D-Cube）](http://www.cnblogs.com/healerkx/archive/2012/01/09/2317579.html)