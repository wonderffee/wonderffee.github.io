---
layout: post
title: "解决终端输命令换行覆盖问题"
date: 2013-08-15 22:24
comments: true
categories: terminal
---

发现终端输入比较长的命令时换行总是出问题，命令超出一行时会出现覆盖提示符的情况，烦不胜烦，今天终于找到问题所在了。
之前为了让终端显示彩色字符，在.bash_profile里增加了这样一个设置：
<!--more-->
{% codeblock lang:bash %}
PS1="\[\e[0;31m\] \[\u@\h \W\]\$ \[\e[m\] "
{% endcodeblock %}

正是这个设置导致命令换行出问题。

对比了一下别人正确的设置，发现我的PS1里多了一对\\[和\\]，也就是中间的那一对\\[和\\]要删掉才行（貌似变成\[和\]也木有问题）。\\[和\\]是用于把非打印字符括起来的（颜色应该是非打印字符吧），别人出的换行问题都是没用\\[和\\]把非打印字符括起来的缘故，我这里却是把不该括的打印字符括了起来也有问题，真是多也不行少也不行。纠正的设置为：
{% codeblock lang:bash %}
PS1="\[\e[0;31m\] \u@\h \W\$ \[\e[m\] "
{% endcodeblock %}

顺便也再调整了一下终端的颜色显示，最终在.bash_probile里做的设置如下：
{% codeblock lang:bash %}
OLOR_BOLD="\[\e[1m\]"
COLOR_DEFAULT="\[\e[0m\]"
export CLICOLOR=1
export GREP_OPTIONS="--color=auto"
PS1='\[\e[01;33m\]\u@\h \W\$\[\e[m\] '
{% endcodeblock %}

参考：   
[终端提示符设置](http://lnote.blogbus.com/logs/106241420.html)    
[Bash长行换行问题](http://linux.fatduck.org/2012/03/bash.html)
