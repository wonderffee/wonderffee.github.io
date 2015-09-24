---
layout: post
title: "Xcode自动生成版本号与根据版本号获取Git提交记录哈希值"
date: 2015-09-24 17:04
comments: true
categories: 
---

相信很多人都用Xcode自动生成版本号，可以省却很多麻烦。这里说的版本号是Build Number，对应plist文件中的CFBundleVersion字段。

其实我最开始看到自动生成版本号的方法是每次自动加1，当时就觉得有一个费解的问题：每修改一次版本号就要改一次plist文件，然后还要提交，团队合作时不是很麻烦而且很容易造成代码提交冲突？
<!--more-->
直到最近我看到这篇文章，才发现了最终的解决方案：[最好的 Xcode 自动生成版本号技术](http://www.cnblogs.com/skyming/p/3978424.html)——不会污染Git记录，也不会造成冲突。

里面使用了下面的命令来获取Git提交次数：

```bash
git rev-list --all|wc -l
```

怎么使用就不多赘述了，看原文就是了，这里就说说原文没说到的一个问题：如何根据版本号得到对应的Git提交记录哈希值？也就是上面自动生成版本号的逆向过程。这适用于一种情况：某个版本出现了崩溃，现在想临时回退到那个崩溃版本的代码。

有人说可以在每次打包的时候打tag，可我终究还是太懒。

首先，在~/.gitconfig里加上下面的代码：

```bash
[alias]
	show-rev-number = !sh -c 'git rev-list --reverse --all | nl | awk \"{ if(\\$1 == "$0") { print \\$2 }}\"'
```

然后你就可以用"git show-rev-number 版本号"命令来获取版本号对应的Git commit哈希值了。

最后根据这个哈希值创建历史分支，你可以进行回退操作了：     
1.git branch (自定义分支名称)　(版本号)　例如：git branch s105 668f074c62406    
2.git checkout (分支)　例如：git checkout s105


###参考：
*   [version control - what is the git equivalent for revision number? - Stack Overflow](http://stackoverflow.com/questions/4120001/what-is-the-git-equivalent-for-revision-number)
*   [最好的 Xcode 自动生成版本号技术 - skyming - 博客园](http://www.cnblogs.com/skyming/p/3978424.html)