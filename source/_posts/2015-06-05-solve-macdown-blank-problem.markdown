---
layout: post
title: "解决MacDown显示空白问题"
date: 2015-06-05 09:34
comments: true
categories: MarkDown, MacDown
---

最近开始用MacDown，喜欢其中的代码高亮功能，边写边预览的感觉很好，顿时觉得可以重新写MarkDown博客了。

但是发现MacDown还是有些不稳定，右栏空白，左栏空白，双栏空白，设置空白的问题都碰到了，要重启多次可能才会好。在[这里](https://github.com/uranusjr/macdown/issues/341)找到了解决方法。

<!--more-->

##How to fix this problem.

###Move the following files to trash:

    
```
    /Users/<your_name>/Library/Preferences/com.uranusjr.macdown.LSSharedFileList.plist  
    /Users/<your_name>/Library/Preferences/com.uranusjr.macdown.plist  
```
###Empty the trash.

If you receive a message that files are busy and cannot be deleted, reboot your OS. Then empty the trash again.

###Consequences

Your preferences will be reset to default; recent files list erased. No other data should be lost (this needs confirmation from @uranusjr )