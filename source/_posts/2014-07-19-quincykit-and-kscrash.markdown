---
layout: post
title: "利用QuincyKit + KSCrash构建自己的Crash Log收集与管理系统"
date: 2014-07-19 10:37
comments: true
categories: iOS
---
其实早就想写这篇文章了，我去年的[一篇文章](http://wonderffee.github.io/blog/2013/08/14/dont-use-symbolicatecrash-to-symbolicate-the-crash-log/)就提到了QunicyKit，一直拖到现在，再不写一年就过去了。

<!--more-->
###前言
我们知道，iOS bug定位是极看重crash log的，目前网上提供了不少crash log收集与管理服务，较有名的有Crashlytics, Flurry, 友盟，可能大部分人也就是使用这个。我这里要说的QuincyKit + KSCrash是一对开源组合，可能没有前者各种高大上的功能，基本功能还是有的，但更偏重于以下使用场合：

1）访问外网不太方便，大部分情况下在内网测试    
2）对出现的crash问题要求快速响应，快速定位     
3）需要自己掌控Crash Report，而不是交给别人     

显而易见，第1条就足以把Crashlytics, Flurry, 友盟诸如此类的排除在外了；    		
关于第2条，我所知道的Flurry显示crash report延迟比较大，至少为6小时，Crashlytics稍微好一些，但是它们的服务器在国外，网页打开也比较慢。这里要额外说的是比较讨厌Crashlytics在程序每次编译时都会上传app binary与dSYM文件，在网络情况较差或app比较大的情况下相当费时。
还有我经历的另一种情况，就是开发与测试人员相隔比较远，比如开发的在5楼，测试的在1楼，在最原始的阶段测试人员发现了崩溃问题，会将测试设备送到4楼让开发人员用Mac解析，想想这效率，不言自明了吧。     
第3点其实比较勉强，crash log又没多少机密可言。

###QuincyKit

相信还是有一些人了解[QuincyKit](https://github.com/TheRealKerni/QuincyKit)的，不过我看到相关的文章比较少。我之前主要参考的是Nico的[《QuincyKit的crashReport框架》](http://www.taofengping.com/2012/12/08/quincykit_crashreport/#.U8PkSI2SzCE)和[《炒冷饭，再提一次QuincyKit》](http://www.taofengping.com/2013/03/29/crash_again/#.U8PkSI2SzCE).

QuincyKit，简而言之，是一个为iOS和Mac OS X提供的程序崩溃报告管理解决方案。关于QuincyKit的介绍大家看Nico的文章就了解得差不多了，它对测试期的应用来说确实是很方便的，不需要提前注册APP Key，不管你有多少个应用，App中集成了QuincyKit的Client端后，只管向Server端发送crash log就行，Server端会自动根据App ID来分类管理。

QuincyKit分Server端和Client端。它的Server端是用php编写的，用一个支持php5.2以上，还有Mysql、Apache的服务器就可以搭建起一个完整的环境，Mac/Linux/Windows系统上应该都可以完成。看似简单，实际上对从没搭建过服务器环境的初学者可能有点麻烦，不用担心，还有XAMPP这个神器来解决大部分麻烦。当初我图简单省事，就是用XAMPP来搭建基本环境的，不过相关的笔记找不到了，这里就没办法贴上了。

另外要说明一下，QuincyKit Server端的Web管理界面比较简陋，比如连按时间排序的功能都没有，不过既然是开源的，就不要要求太高，会PHP的完全可以自己订制更高级的功能。


###KSCrash

这里要说的是我只使用了QuincyKit的Server端，Client端我选择了[KSCrash](https://github.com/kstenerud/KSCrash)，可能有很多人对KSCrash对比较陌生，这也不奇怪，在国内就没见到有人介绍KSCrash。为什么是KSCrash呢，个中原因，且听我慢慢道来：

1）如果我没记错的话，QuincyKit client端生成的crash报告与原生crash报告相比总是缺少最关键的那一行，而KSCrash客户端生成的crash报告会把这关键的一行放在最后一行，并提供一些额外的信息，非常有利于问题定位。大部分情况下，我只看这最后一行就能定位到问题所在。    
2）QuincyKit在2013年初基本就停止更新了，而KSCrash目前仍旧是持续更新的，对于我们来说最重要是Client端的更新，比如考虑未来支持Swift的可能性。而Server端主要是用来管理crash log，免费开源的QuincyKit足够对付使用。    
3）KSCrash客户端生成的crash报告在大部分情况下都不需要dSym符号文件你就可以看到函数名，问题比较明显的话很快就能得到定位。但是默认显示的行号还是不对的，如果需要具体行号，还得利用dSym符号文件解析crash报告才行。（这点似乎QuincyKit客户端也支持的）    

最关键是第1点，我想会不会有人因为这放弃使用了QuincyKit。

根据KSCrash的官网上的说明，它能处理以下几种类型的crash：

- Mach kernel 异常    
- Fatal signals    
- C++ 异常    
- Objective-C异常    
- 主线程死锁 (实验性质)    
- 自定义崩溃（脚本语言）    

KSCrash只有一个client端，它能对接Hockey、QuincyKit、Victory等Server端，也能将生成的crash log通过Email的方式发送。

经过我的测试，使用KSCrash目前主要有以下两个问题：      
1) 在部分情况下会造成死锁（deadlock）crash，这通常是应用在做比较耗时的操作时出现的，如果发生这样的问题，你应该尽量优化你的APP，避免耗时操作（可能是我打开了死锁检测的功能，目前这个功能是Unstable Features）     
2) 如果发生crash后重启应用时收集crash报告的服务器不可达，则之前的crash报告就会被废弃，即使重新恢复网络，仍然不会重新发送。     

如果有人介意这些问题，因KSCrash是开源的，可以自行修改。

App发布时，还是建议大家使用Flurry、Crashlytics或者友盟，毕竟你面对的可能是数以万计甚至更多的用户。你可以在你的应用中增加一个开关，测试期打开开关使用KSCrash上报crash log，发布前关闭开关，使用Crashlytics或其它在线crash report工具。万万要记得在自己的checklist里加上一条开关核对，以免忘记。


###Crash Log自动解析

关于QuincyKit还有一个解析端，或者说是Mac端，因为crash log的解析是必须在Mac上进行的。要解析QuincyKit Server端收集到的crash log，就必须在解析端的Mac电脑上执行一个symbolicate.php脚本。它做的事情实际上就是将QuincyKit上未解析过的crash log下载到本地，批量进行解析，然后再批量上传回去。你可以启动一个定时任务定时去执行这个脚本，就不用每次都手动执行了。

但是有一个问题，如果你想每次都能解析成功，你的Mac上需要提前拥有与crash log对应的.app文件、.app.dSYM文件的索引。这个索引可以通过mdimport命令来实现（mdimport的介绍可参考[这里](http://wonderffee.github.io/blog/2013/08/14/dont-use-symbolicatecrash-to-symbolicate-the-crash-log/)）。不过肯定还是有人觉得手动执行mdimport命令进行索引挺麻烦的，最好的办法还是用脚本将各个流程串通起来自动化实现。

我能想到的一个自动化流程是：    
(1) 自动构建版本，生成ipa文件和dSYM.zip文件    
(2) 解析端通过脚本拿到ipa文件和dSYM.zip文件，然后copy到指定文件夹，解压，执行mdimport     
(3) 在解析端的Mac电脑上开启一个定时任务，定时执行symbolicate.php    

经历这3步，就可以保证你在QuincyKit web网页上永远看到的是解析后的crash log。

做到了这一切，如果你经历过手动执行symbolicatecrash命令来解析crash log的阶段，就知道一个是天上，一个是地下了。

###后记

原谅我没有贴图，因为我都是在公司里搭的系统，家里没有。在网上就找到一张QuincyKit的web管理界面，供参考

{% img /images/quincykit_web.jpg %}

搜索资料时还有意外发现，《[Doutzen: Local symbolication for QuincyKit](http://nielsmouthaan.nl/doutzen/)》一文的作者做了一个Mac应用来完成解析端的工作，将配置简单化，并实现了定时自动解析。不过根据评论，应该是不兼容Lion系统，估计更不会兼容最新的10.9/10.10了，好在代码是开源的，会Mac开发的稍作修改就可以拿来为己所用了。

还有人用Rails写了一个类QuincyKit的Server端，提供在线服务网站holdbug.com，但是代码好像不开源，网站也打不开了，估计是停止支持了，链接见[这里](http://ju.outofmemory.cn/entry/37054)。


###参考：     
- [QuincyKit](https://github.com/TheRealKerni/QuincyKit)
- [KSCrash](https://github.com/kstenerud/KSCrash)   
- [QuincyKit的crashReport框架](http://www.taofengping.com/2012/12/08/quincykit_crashreport/#.U8PkSI2SzCE)        
- [炒冷饭，再提一次QuincyKit](http://www.taofengping.com/2013/03/29/crash_again/#.U8PkSI2SzCE)        
- [The battle of the iOS crash reporters](http://www.jeremyfuller.net/the-battle-of-the-ios-crash-reporters/comment-page-1/)     
- [Doutzen: Local symbolication for QuincyKit](http://nielsmouthaan.nl/doutzen/)     
- [【开源】最近用 rails 做了一个 ios crash 的收集系统](http://ju.outofmemory.cn/entry/37054) 