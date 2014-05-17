---
layout: post
title: "unrecognized selector sent to instance问题之诱敌深入关门打狗解决办法"
date: 2014-05-17 11:03
comments: true
categories: iOS
---

前不久在微博上看到一篇文章，[《UNRECOGNIZED SELECTOR SENT TO INSTANCE 问题快速定位的方法》](http://blog.objcc.com/unrecognized-selector-sent-to-instance/) 其中讲了iOS unrecognized selector sent to instance问题的快速定位方法，方法是不错的，但是实际测试发现文中的方法并非万能，从我自身的经历以及文中的评论看都有不能解决的情况。
<!--more-->
出现unrecognized selector sent to instance问题，大部分是因为对象被提前释放，指针变成野指针，还有一种情况是本身就是野指针，如声明一个局部对象，没有初始化就直接调用。定位难的原因是你知道这个野指针指到哪个类了，但是不知道是哪里产生了野指针。如果一个正常的对象调用一个不存在的方法，也会给出这个提示，不过这种情况下Xcode会直接给出crash的代码行，不存在定位难的问题。

我遇到这个问题的情况是这样的：写的代码一直在iOS7下进行调试，运行得好好的，最近想测试一下iOS6的兼容性，结果登录成功后就会产生crash，提示[NewsViewController size] unrecognized selector sent to instance，看到这个问题当时真是相当莫名其妙，NewsViewController无论如何都不可能有size这个方法，是什么让NewsViewController调用这个方法呢？

在Xcode中用size关键词搜索所有调用size的地方一个个排除？别逗了，代码里多的是。想象一下，你在iOS7下写好了全部代码，然后在iOS6下测试兼容性时出现此问题，面对茫茫如海的代码，足够让你望洋兴叹了，一个个去找，费不起那功夫。

想起来上面那篇文章中的方法，结果是毫无帮助，下断点无效。

只得再另想办法。要快速定位问题代码行，主要思路还是得下断点，还有没有别的办法下断点呢？这个时候可就要在“unrecognized selector sent to instance”的提示上做文章了，这个提示的实际意义是某个对象调用了不存在的方法。不妨逆向思考一下，既然它没有，我如果给它加上一个呢？这不下断点的机会就来了——所谓诱敌深入，关门打狗，不过如此。

于是，我就在NewsViewController中加了一个这样的方法：
```objc
- (CGSize)size{
NSLog(@"test");
return CGSizeZero;
}
```
在其中的NSLog行加上断点，运行工程，果然就找到了调用该方法的代码行，问题迎刃而解。

出错的代码也贴一下吧，简化一下大概就是下面这样的:
```objc
- (void)test{
UIImage *imgNormal, *imgSelected;
NSLog(@"imagNormal width is %f", imgNormal.size.width);
}
```

问题出在NSLog的那一行，很显然，这就是没有初始化的局部对象在实际访问时出错，系统认为它是NewsViewController对象， 不再属于UIImage类了。

需要注意的是，上面的代码你拿过去并不一定能复现同样的问题，可能就不会发生crash了。这里只是提供另一种解决思路，希望对遇到此问题的人有所帮助。

