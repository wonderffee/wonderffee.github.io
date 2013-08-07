---
layout: post
title: "objective-c的category并非不能增加新属性"
date: 2013-08-07 22:46
comments: true
categories: 
---

上一篇文章中应用到了hidesBottomBarWhenPushed属性，关于这个属性，如果你够仔细就可以发现它是在UINavigationController.h文件中定义的，而不是在UIViewController中定义的。这是利用了objective-c的category特性，原代码如下：
<!--more-->

{% codeblock Here's Code lang:objc %}
@interface UIViewController (UINavigationControllerItem)

@property(nonatomic,readonly,retain) UINavigationItem *navigationItem; // Created on-demand so that a view controller may customize its navigation appearance.
@property(nonatomic) BOOL hidesBottomBarWhenPushed; // If YES, then when this view controller is pushed into a controller hierarchy with a bottom bar (like a tab bar), the bottom bar will slide out. Default is NO.
@property(nonatomic,readonly,retain) UINavigationController *navigationController; // If this view controller has been pushed onto a navigation controller, return it.

@end
{% endcodeblock %}

category是一个很棒的objective-c特性，可以为已经存在的类增加方法，而不需要增加一个子类。不过你也许会模糊地记得category是不能给一个类增加属性的(除非借助associative)，是不是对这里的category能给一个类增加新的属性非常奇怪呢？

其实不是这样的，objective-c中的category只是不能增加实例变量，这里添加的hidesBottomBarWhenPushed属性只是添加了setter和getter方法的声明，在对应的m文件中应该是实现了这两个方法并调用了其它的private变量或利用了runtime机制，相关讨论可以见这里：[类别不是应该只能添加方法吗？类别现在能直接添加属性了？](http://www.cocoachina.com/bbs/read.php?tid=132558)。用category添加了属性如何实现对应的set、get方法可以参考这个：[让Category支持添加属性与成员变量](http://www.cnblogs.com/wupher/archive/2013/01/05/2845338.html)。

有一个很意思的问题，从xcode4.5开始只要声明了property就可以自动生成实例变量，但是如果是在一个类的category类别中添加新的property，会自动生成对应的实例变量吗，有兴趣的可以自己动手试试。