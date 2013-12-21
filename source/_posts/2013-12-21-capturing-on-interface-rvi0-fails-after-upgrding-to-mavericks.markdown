---
layout: post
title: "capturing on interface rvi0 fails after upgrding to Mavericks "
date: 2013-12-21 22:27
comments: true
categories: tools
---

MacBook Pro升级至Mavericks后，用Wireshark在rvi0上抓取iOS设备的流量包就出了问题，抓包不能正常显示，白茫茫的一片，没有有效抓包信息。

google了一下，有人遇到同样的问题，看来是Mavericks修改了一些东西。
<!--more-->
找到了不完全解决办法：

1、在Wireshark的Edit菜单中打开Preferences->Protocols->DLT_USER

2、选择Edit——New——User 2(DLS=149), payload protocol项输入eth, Header size项输入108， 点击OK保存。

再重新抓包就可以了。

之所以说不完全，是因为抓包不能设置端口过滤，一旦设置端口过滤（比如增加port 8000过滤）的时候wireshark会自动退出，真是奇哉怪也。

测试了一下，是在Link-layer heaer为Pocket Tap（rvi0的默认设置）时增加端口过滤会闪退，而修改为Raw IP不会闪退，但这样修改的话又完全抓不了包,囧里个囧。。。。

实在不行可以用tcpdump抓包：
```bash
sudo tcpdump -i rvi0 -n -s 0 -w capture.pcap port 8000
```

参考：     
[Mavericks - can not capture from iPhone using RVI](http://ask.wireshark.org/questions/26524/mavericks-can-not-capture-from-iphone-using-rvi)