---
layout: post
title: "通过实例来理解position与anchorPoint"
date: 2013-10-14 22:58
comments: true
categories: iOS
---
[上一篇文章](http://wonderffee.github.io/blog/2013/10/13/understand-anchorpoint-and-position/)写了一些对position与anchorPoint的理解，这次就拿一些实例来加深印象。文中的例子来自别人的一篇[博文](http://www.cnblogs.com/yyh123/p/3345097.html)，例子是不错的，但是自己刚开始也没完全搞明白，现在完全弄懂了，在这里借用一下并加以扩展，希望对看到的人有所帮助。
<!--more-->

先把之前的结论贴出来：  
1、position是layer中的anchorPoint在superLayer中的位置坐标。  
2、互不影响原则：单独修改position与anchorPoint中任何一个属性都不影响另一个属性。  
3、frame、position与anchorPoint有以下关系：  
{% codeblock lang:objc %}
frame.origin.x = position.x - anchorPoint.x * bounds.size.width；  
frame.origin.y = position.y - anchorPoint.y * bounds.size.height；  
{% endcodeblock %}

###1.创建一个CALayer，添加到控制器的view的layer中
{% codeblock lang:objc %}
CALayer *myLayer = [CALayer layer];
// 设置层的宽度和高度（100x100）
myLayer.bounds = CGRectMake(0, 0, 100, 100);
// 设置层的位置
myLayer.position = CGPointMake(100, 100);
// 设置层的背景颜色：红色
myLayer.backgroundColor = [UIColor redColor].CGColor;

// 添加myLayer到控制器的view的layer中
[self.view.layer addSublayer:myLayer];
{% endcodeblock %}

第5行设置了myLayer的position为(100, 100)，又因为anchorPoint默认是(0.5, 0.5)，所以最后的效果是：myLayer的中点会在父层的(100, 100)位置。

My Note:根据anchorPoint默认为(0.5, 0.5)以及结论3中的的公式，可以得到frame.origin为(50, 50),所以myLayer在图中左上角的位置应为(50, 50)。见下图:  

{% img /images/anchorPointInstance/1.png %}

###2.若将anchorPoint改为(0, 0)
{% codeblock lang:objc %}
myLayer.anchorPoint = CGPointMake(0, 0);
{% endcodeblock %}

MyNote:根据结论2，修改anchorPoint不影响position，则frame.origin需要重新计算，如下：
{% codeblock lang:objc %}
frame.origin.x = 100 - 0 * 100 = 100;
frame.origin.y = 100 - 0 * 100 = 100;
{% endcodeblock %}
这样，myLayer的左上角就会就会移动到(100, 100)的位置。见下图：  
{% img /images/anchorPointInstance/2.png %}

###3.若将anchorPoint改为(1, 1)
{% codeblock lang:objc %}
myLayer.anchorPoint = CGPointMake(1, 1);
{% endcodeblock %}

MyNote:同上，直接重新计算frame.origin，会得到frame.origin为(0, 0)，myLayer左上角移动到(0, 0)的位置，见图：  
{% img /images/anchorPointInstance/3.png %}

###4.将anchorPoint改为(0, 1)
{% codeblock lang:objc %}
myLayer.anchorPoint = CGPointMake(0, 1);
{% endcodeblock %}
MyNote:类似地，直接重新计算frame.origin，会得到frame.origin为(100, 0)，myLayer左上角移动到(100, 0)的位置，见图：    
{% img /images/anchorPointInstance/4.png %}

###5.设置frame与bounds的区别
先看代码1：
{% codeblock lang:objc %}
CALayer *myLayer1 = [CALayer layer];
myLayer1.bounds = CGRectMake(0, 0, 100, 100);
myLayer1.anchorPoint = CGPointZero;
myLayer1.backgroundColor = [UIColor redColor].CGColor;
[self.view.layer addSublayer:myLayer1];
{% endcodeblock %}

再看代码2：
{% codeblock lang:objc %}
CALayer *myLayer2 = [CALayer layer];
myLayer2.frame = CGRectMake(0, 0, 100, 100);
myLayer2.anchorPoint = CGPointZero;
myLayer2.backgroundColor = [UIColor redColor].CGColor;
[self.view.layer addSublayer:myLayer2];
{% endcodeblock %}

区别在于第二行代码一个使用bounds,一个使用frame, 猜猜myLayer1和myLayer2左上角的位置会相同吗？

你可能觉得是相同的，但其实不同，myLayer1左上角在(0, 0)点，myLayer1却不是预期的(0, 0)点，而是在(50, 50)点，也就是例1中的图，为什么？有人可能还会说我明明想将myLayer2左上角放在原点，怎么就不是呢？

关键在于用的是frame。对于一个还没有superLayer的layer来说，position和anchorPoint都是有默认值的，分别为(0, 0)和(0.5, 0.5),如果对该layer设置了frame，因为anchorPoint还是保持默认值不会变化，只能是position随之变动。所以根据3中的公式，position的计算如下：
{% codeblock lang:objc %}
position.x = 0 + 0.5 * 100 = 50
position.y = 0 + 0.5 * 100 = 50
{% endcodeblock %}

position被确定为(50, 50)，接着在代码2中又修改anchorPoint为(0, 0), 这个时候不影响position的值，只能是frame.origin被修改，也就是变成下面的：
{% codeblock lang:objc %}
frame.origin.x = 50 - 0 * 100 = 50;
frame.origin.y = 50 - 0 * 100 = 50;
{% endcodeblock %}

所以对一个已经确定frame的layer来说，再修改anchorPoint就会修改layer的位置，如果只想修改anchorPoint而不改变layer的位置，就用代码1的方式来进行，或者把代码2中设置frame与anchorPoint的代码顺序调换一下，就能解决问题。

要记住的就是设置frame会隐式修改position，而默认的anchorPoint从来不会被隐式修改，只能被显式修改。

####参考
* [position和anchorPoint](http://www.cnblogs.com/yyh123/p/3345097.html)
* [Changing my CALayer's anchorPoint moves the view](http://stackoverflow.com/questions/1968017/changing-my-calayers-anchorpoint-moves-the-view)  