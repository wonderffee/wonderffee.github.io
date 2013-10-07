---
layout: post
title: "一些MacOS技巧"
date: 2013-10-07 08:48
comments: true
categories: 
---

早早就醒了，天气又不好，无事可做，整理一下自己在使用Mac过程中用到的一些技巧吧，比较杂乱，以后不定期更新。

###如何在命令行中使用Sublime Text
终端中输入下面的命令回车即可，之后就可以使用subl filename打开文件了。
<!--more-->
{% codeblock lang:bash %}
sudo ln -s /Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl /usr/bin/subl
{% endcodeblock %}

###如何查看当前dns设置
cat /etc/resolv.conf

###如何在命令行中快速修改DNS
{% codeblock lang:bash %}
sudo networksetup -setdnsservers Wi-Fi 199.91.73.222 178.79.131.110
{% endcodeblock %}

###如何在命令行中快速得到iOS设备的UDID
{% codeblock lang:bash %}
ioreg -w 0 -rc IOUSBDevice -k SupportsIPhoneOS | sed -n 's/.*USB Serial Number[^0-9a-z]*\([0-9a-z]*\).*/\1/p'
{% endcodeblock %}

###如何快速创建HTTP服务器用于文件共享
一行命令即可，在命令行输入“python -m SimpleHTTPServer 8888”，就可以创建以当前python命令运行的目录为根目录的HTTP服务器了。在浏览器打上http://xx.xx.xx.xx:8888/ 就能访问到python所运行的当前目录了，下载文件很方便。    
——**在微博上看到的这个技巧，高端大气上档次！**

###如何解决iTunes下载时出现"无法完成您的itunes store的请求,发生未知错误(-50)"问题
打开iTunes -- 编辑 -- 偏好设置 -- 家长控制 -- iTunes Store这一项勾选（把 允许访问iTunes U 这一项也勾选）-- 确定。 这时iTunes Store会自动访问iTunes U，关闭iTunes。 重开iTunes，回到家长控制把之前勾选的两项取消，回iTunes Store重新登录，-50问题就解决了！