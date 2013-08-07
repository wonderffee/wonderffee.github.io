---
layout: post
title: "自定义Tabbar实现push动画隐藏效果"
date: 2013-08-07 22:45
comments: true
categories: 
---


在之前的一篇文章([链接](http://wonderffee.github.io/blog/2013/08/05/simulate-weico-custom-tabbar/))中我写到了没有用UITabbarController来实现一个自定义Tabbar，当然功能也简陋了点。注意到在Weico或微信中的自定义tabbar有一个这样的功能：push到下一个页面时tabbar会被自动隐藏，下面我就来说说如何使我前面做的自定义tabbar也能实现隐藏。
<!--more-->

如果是原生的tabbar，这个功能实现很容易。在iOS中，每个UIViewController都有一个属性hidesBottomBarWhenPushed，每次push一个viewController时，设置viewController. hidesBottomBarWhenPushed=YES就可以自动实现前面所说的隐藏功能。但是前提是必须使用UITabbarController，我这里实现的自定义tab bar完全没有使用UITabbarController，那就要多费点心。

如果要实现自定义tabbar在每次push viewController时隐藏，很显然我们需要push时有事件能够通知自定义tab bar隐藏。如果你熟悉UINavigationController的push流程的话，应该就知道我们可以让UINavigationController执行
push时调用navigationController: willShowViewController:方法来触发通知，前提是要遵守UINavigationControllerDelegate协议。
由于hidesBottomBarWhenPushed是每个UIViewController都有的属性，我们姑且还是把它用上。代码如下：

{% codeblock Here's Code lang:objc %}
- (void)navigationController:(UINavigationController *)navigationController
  willShowViewController:(UIViewController *)viewController
                animated:(BOOL)animated
{
    if (viewController.hidesBottomBarWhenPushed) {
        self.tabBar.hidden = YES;
    } else {
        self.tabBar.hidden = NO;
    }
}
{% endcodeblock %}

这样的实现是比较简单的。对比weico或微信iPhone应用的自定义tab bar push隐藏行为，你就会发现它们有一个自然的过滤动画来实现隐藏，而且与viewController的push动画同步，这是上面的代码做不到的。如果要实现这个动画，就需要对self.tabbar设置frame的过渡动画，代码如下：

{% codeblock Here's Code lang:objc %}
- (void)navigationController:(UINavigationController *)navController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    if (viewController.hidesBottomBarWhenPushed)
    {
        [self hideTabBar];
    }
    else
    {
        [self showTabBar];
    }
}

- (void)hideTabBar {
    if (!tabBarIsShow)
    { //already hidden
        return;
    }
    [UIView animateWithDuration:0.35
                     animations:^{
                         CGRect tabFrame = tabBar.frame;
                         tabFrame.origin.x = 0 - tabFrame.size.width;
                         tabBar.frame = tabFrame;
                     }];
    tabBarIsShow = NO;
}

- (void)showTabBar {
    if (tabBarIsShow)
    { // already showing
        return;
    }
    [UIView animateWithDuration:0.35
                     animations:^{
                         CGRect tabFrame = tabBar.frame;
                         tabFrame.origin.x = CGRectGetWidth(tabFrame) + CGRectGetMinX(tabFrame);
                         tabBar.frame = tabFrame;
                     }];
    tabBarIsShow = YES;
}
{% endcodeblock %}

上面代码中的0.35秒这个时间保证了与tabbar的隐藏动画与viewController的push动画同步，基本上可以实现以假乱真的效果。

代码链接：[https://github.com/wonderffee/idev-recipes/tree/master/CustomTabBar
](https://github.com/wonderffee/idev-recipes/tree/master/CustomTabBar
)