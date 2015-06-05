---
layout: post
title: "使用MacDown的一些笔记"
date: 2015-06-05 09:34
comments: true
categories: MarkDown, MacDown
---

最近开始用MacDown，喜欢其中的代码高亮功能，边写边预览的感觉很好，顿时觉得可以重新写MarkDown博客了。



但是发现MacDown还是有些不稳定，右栏空白，左栏空白，双栏空白，设置空白的问题都碰到了，要重启多次可能才会好。在[这里](https://github.com/uranusjr/macdown/issues/341)找到了解决方法，但好像只能部分解决。

<!--more-->

##解决空白显示问题

###Move the following files to trash:

    
```
    /Users/<your_name>/Library/Preferences/com.uranusjr.macdown.LSSharedFileList.plist  
    /Users/<your_name>/Library/Preferences/com.uranusjr.macdown.plist  
```
###Empty the trash.

If you receive a message that files are busy and cannot be deleted, reboot your OS. Then empty the trash again.

###Consequences

Your preferences will be reset to default; recent files list erased. No other data should be lost (this needs confirmation from @uranusjr )

##命令行启动MacDown
之前用Sublime Text可以很方便地用命令行启动。MacDown当然也有，并且我发现了一个很简便的命令行启动软件的方法，应该对任何Mac软件都适用吧。

代码：

```bash
sudo echo "open -a MacDown $*" > /usr/local/bin/macdown
sudo chmod +x /usr/local/bin/macdown
```
方法来源于此：[Added command line utility. by sorig · Pull Request #35 · uranusjr/macdown](https://github.com/uranusjr/macdown/pull/35)。

实际使用发现用macdown命令打开文件时，并没有打开文件。vi /usr/local/bin/macdown一看，发现少了一个$*，加上就好了。