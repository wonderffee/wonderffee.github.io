---
layout: post
title: "iOS抓包利器Charles"
date: 2013-07-13 11:21
comments: true
categories: 
---


看唐巧的[分析支付宝客户端的插件机制](http://blog.devtang.com/blog/2013/06/23/alipay-plugin-mechanism/)一文发现他使用了抓包工具Charles，想起去年有人给我推荐过这个工具，但是当时我觉得WireShark就够用了就没尝试。这次看到又有人使用Charles我就重视起来了，Charles到底有什么好？

搜了一下，发现大多数使用者都是将Charles作为移动端抓包工具使用的，这样就意味着我们可以用Charles来截取app所发出的网络请求来进行分析，[分析支付宝客户端的插件机制](http://blog.devtang.com/blog/2013/06/23/alipay-plugin-mechanism/)一文就是这么用的。WireShark显然做不到这一点，优势一下子就体现出来了。

在Mac上安装Charles后，启动Charles，首先弹出一个框提示是否允许Charles有自动修改网络设置的权限，选择允许后出现Charles主界面。Charles主界面左侧有Structure和Sequence，你会发现会发现Structure这一栏里会逐步出现当前我的mac正在请求的链接，也就是说Charles一启动就自动进行抓包了。不过遗憾的是Structure栏里没有过滤选项，意味着你不能过滤特定网站。切换到Sequence栏，这个就容易懂了，按时间顺序来排列的，与WireShark一致。下方的Filter可以过滤，而是还是实时过滤的，这一点就比WireShark强多了。

 {% img /images/Structure.png %}

 {% img /images/Sequence.png %}

如何在Mac上用Charles远程抓iPhone上app的网络请求呢？方法相当简单，下面就提供了HTTP和HTTPS抓包的操作步骤，简单几步就搞定了。

###HTTP抓包
*   打开Charles程序
*   查看Mac电脑的IP地址，如192.168.1.7
*   打开iOS设置，进入当前wifi连接，设置HTTP代理Group，将服务器填为上一步中获得的IP，即192.168.1.7，端口填8888
*   iOS设备打开你要抓包的app进行网络操作
*   Charles弹出确认框，点击Allow按钮即可

###HTTPS抓包
*   下载Charles证书<http://www.charlesproxy.com/ssl.zip>，解压后导入到iOS设备中（将crt文件作为邮件附件发给自己，再在iOS设备中点击附件即可安装；也可上传至dropbox之类的网盘，通过safari下载安装）
*   在Charles的工具栏上点击设置按钮，选择Proxy Settings…
*   切换到SSL选项卡，选中Enable SSL Proxying，别急，选完先别关掉，还有下一步
*   这一步跟Fiddler不同，Fiddler安装证书后就可以抓HTTPS网址的包了，Charles则麻烦一些，需要在上一步的SSL选项卡的Locations表单填写要抓包的域名和端口，点击Add按钮，在弹出的表单中Host填写域名，比如填api.instagram.com，Port填443

我简单试用了一下Charles的远程抓包功能，发现Charles比WireShark还有一个优势是能对JSON数据(在JSON Text栏)进行解析，从而让我们可以更直观地查看JSON串信息(在JSON 栏)。此外Charles对中文支持比较好，JSON串中的中文信息一般会显示为一长串的\ug开头的字符，解析之后就能显示出中文了。平常总头痛Wireshark对中文支持不好，用Charles就完全没有这个问题了。

参考资料：   
[使用Charles远程调试iOS移动应用](http://larryhou.github.io/blog/2012/11/05/remote-debug-with-charles-proxy/)     
[mac下的抓包工具Charles](http://ju.outofmemory.cn/entry/32837)