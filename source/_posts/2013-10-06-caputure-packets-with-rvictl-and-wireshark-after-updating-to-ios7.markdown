---
layout: post
title: "升级iOS7后利用rvictl和wireshark抓包失效?"
date: 2013-10-06 11:31
comments: true
categories: iOS
---

最近把一台设备升级到iOS7后，利用rvictl和wireshark抓包发现抓不了，无意中发现在装有xcode5的机器上可以抓包，看来rvictl与xcode是绑定的，升级到最新的iOS7后，必须要装上最新的xcode5版本才能抓包。

使用rvictl有一个前提是要获取设备的UDID，看网上不少教程都是从xcode中获取UDID，步骤相当繁琐，快速获取UDID用命令行才是王道，果然不出所料，很快就找到了三种命令行快速得到iOS设备的UDID方法，如下：
<!--more-->

方法1：速度最快
{% codeblock lang:bash %}
ioreg -w 0 -rc IOUSBDevice -k SupportsIPhoneOS | sed -n 's/.*USB Serial Number[^0-9a-z]*\([0-9a-z]*\).*/\1/p'
{% endcodeblock %}

方法2：不知道为什么不起作用了，因为iOS7的缘故？
{% codeblock lang:bash %}
system_profiler SPUSBDataType | sed -n -e '/iPhone/,/Serial/p' | grep "Serial Number:" | awk -F ": " '{print $2}'
{% endcodeblock %}

方法3: 
{% codeblock lang:bash %}
system_profiler SPUSBDataType | grep "Serial Number:.*" | sed s#".*Serial Number: "##
{% endcodeblock %}

不过注意的是，用上面的命令得到的UDID并不一定是唯一的，比如我在MacBook Pro上就得不到唯一的UDID。

以下转载一下用rvictl和wireshark进行抓包的方法：

RVI(Remote Virtual Interface）是在iOS5中开始添加的，利用这个工具，在不需要开代理，也不需要越狱的情况下就可以抓到iOS设备上所有的包，所需要的一台装有Mac OS X的电脑以及USB数据线。我发现Android还没有类似的工具，iOS在这方面就方便多了。

基本的方法就是把设备通过USB连上mac上。然后为这台设备安装RVI，这个虚拟的在Mac上的网卡，就代表这台ios设备的使用网卡。然后在mac上跑抓包的工具，定位到这个虚拟的网卡上，来抓包。

(1)安装RVI，需要使用rvictl工具，以下步骤在mac的终端中操作：    
{% codeblock lang:bash %}
$ # First get the current list of interfaces.
$ ifconfig -l
lo0 gif0 stf0 en0 en1 p2p0 fw0 ppp0 utun0
$ # Then run the tool with the UDID of the device.

$ rvictl -s 74bd53c647548234ddcef0ee3abee616005051ed
Starting device 74bd53c647548234ddcef0ee3abee616005051ed [SUCCEEDED]

$ # Get the list of interfaces again, and you can see the new virtual
$ # network interface, rvi0, added by the previous command.
$ ifconfig -l
lo0 gif0 stf0 en0 en1 p2p0 fw0 ppp0 utun0 rvi0
{% endcodeblock %}

(2)安装成功后，此时其实可以用任何抓包工具来抓取。包括wireshark等。因为这时就会看到一个rvi0的网卡。不过今天我们介绍的是通过tcpdump来搞。

在终端中输入如下命令：
{% codeblock lang:bash %}
sudo tcpdump -i rvi0 -n -s 0 -w dump.pcap tcp
{% endcodeblock %}

解释一下上面重要参数的含义：

-i rvi0 选择需要抓取的接口为rvi0（远程虚拟接口）    
-s 0 抓取全部数据包    
-w dump.pcap 设置保存的文件名称    
tcp 只抓取tcp包    

当tcpdump运行之后，你可以在iOS设备上开始浏览你想抓取的App，期间产生的数据包均会保存到dump.pcap文件中，当想结束抓取时直接终止tcpdump即可。然后在mac中找到dump.pcap文件。用wireshark打开就ok。

(3)去掉RVI这个虚拟网卡，使用下面的命令：   
{% codeblock lang:bash %} 
$ rvictl -x 74bd53c647548234ddcef0ee3abee616005051ed    
Stopping device 74bd53c647548234ddcef0ee3abee616005051ed [SUCCEEDED]   
{% endcodeblock %}

整个流程就是这样的。自己动手操作一下吧。。

再详细的细节，请找一下苹果官方的资料：Technical Q&A QA1176。

参考链接：    
[Mac上命令行获取iPhone/iPad的Identifier（UUID） 的方法](http://blog.csdn.net/hursing/article/details/8688868)    
[一行命令 得到iOS设备的UDID](http://b.imi.im/post/121)    
[未越狱ios设备的抓包方法](http://fanliugen.com/?p=351)    

