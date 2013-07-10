---
layout: post
title: "搭建基于Github的Octopress博客"
date: 2013-07-09 23:18
comments: true
categories: 
---

我无意像其它人一样写一个大而全的教程，这样的教程在网上已经有很多，你只需要参考下面任何一个教程都可以得到满意的答案。

*   [Octopress官方文档](http://octopress.org/docs/)           
*   [象写程序一样写博客：搭建基于github的博客](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)
*   [在github上搭建octopress博客](http://biaobiaoqi.me/blog/2013/03/21/building-octopress-in-github-mac/)
*   [利用Octopress在github Pages上搭建个人博客](http://easypi.github.io/blog/2013/01/05/using-octopress-to-setup-blog-on-github/)
*   [使用Octopress + Github管理blog](http://ishalou.com/blog/2012/10/15/how-to-use-octopress/)      
其中前面三个链接是我主要参考的。

这里，我主要想记录我遇到的一些问题，以期望能对那些遇到类似问题的人有些许帮助。当然，如果你在操作之前先看到了这篇文章，那也不妨先看看再更有底气地动手。

###Github repository设置
基于Github的博客其实是利用了Github Pages支持html上传的功能来实现的，那么怎么开通Github Pages呢？答案很简单，在你的Github上创建一个名为username.github.io的repository，然后上传html就可以了。上传hmtl就是后面提到的rake deploy命令负责的，先不用管，这里说说repository的命名。

多数教程建议在GitHub上建立名为username.Github.com的repository（其中username要替换成Github真实用户名），但是我根据Octopress教程操作时，发现要求提供的Git地址是git@github.com:username/username.github.io.git，当时我就迷惑了，到底是.com还是.io？之后才发现.com这个后缀是Github早期推荐使用的，现在推荐使用.io格式，为了兼容起见，Github还是让username.github.com能够访问，不过是让它自动指向username.github.io而已。因此为了避免疑惑，对初次使用者还是建议使用username.github.io来命名自己的repository.

如果你还是一位Git初学者，在建repositionary时最好不要勾选生成README.md文件，说来惭愧，由于本人对Git还停留在准入门阶段，一时手贱结果让这个东东在后面给我造成了一些麻烦。如果你是Git初学者就信我这一回，不要勾选。


###Ruby安装失败问题
准备条件中有一条要有ruby 1.9.3环境，我的MacOS系统(10.8.1)原生自带ruby，就是版本是1.8.7低了一点，需要通过rvm来升级到1.9.3版本。需要注意的是ruby现在已经是2.0版本了，不确定用最新版本的ruby能否安装成功，因此不要贸然尝试将ruby更新到最新版本。    

我下载rvm后安装ruby 1.9.3，执行命令：rvm install 1.9.3，遇到下面的问题   
There was an error while trying to resolve rubygems version for 'latest'.     
Halting the installation.     
不明原因，解决方法是重试：rvm reinstall 1.9.3     
参考：<http://misheska.com/blog/2013/06/16/using-rvm-to-manage-multiple-versions-of-ruby/>     

###Github SSH key
Octopress博客的创建过程中需要用到Github SSH key，生成Github SSH key的方法可以参考Github官方的[Generating SSH Keys](https://help.github.com/articles/generating-ssh-keys)教程

###访问博客时显示404的问题
我按照别人博客教程里提供的方法弄完了，去看我的博客，发现给了一个404的页面。有人说要等十几分钟才会正常，结果我等了一个多小时还是原样。是怎么回事呢，最后才发现我漏输了一个"rake deploy"命令，导致我的本地博客根本就没有发布到Github上去，真是大意。  

这还得从名为username.github.io的repository的构成说起，每个在Github上发布成功的博客，在username.github.io这个repository里都会有两个分支，分别为master和source。其中master分支存放博客正常显示所需要的静态文件（也就是rake generate生成的），source分支下是整个博客的全部源码。"rake deploy"命令的主要作用就是把本地的_deploy目录下面的内容push到远程的master分支，如果忘记执行这条命令，master分支下就没有内容，访问博客自然也就会出现404错误了。


###关于博客配置与MarkDown学习
Github博客的配置主要在_config.yml这个文件中进行，各个教程也都做了简短介绍，比如去掉一些不用的侧边栏，增加分享与评论功能，但仅限于此。

在Github上写博客，MarkDown是必须的技能。基本上每个提供搭建Github博客教程都会让你去看MarkDown的语法教程或的网上介绍文章，比如[这个](http://daringfireball.net/projects/markdown/)，还有[这个](http://wowubuntu.com/markdown/index.html)。

但是我想说最好的教程就是各位大牛自己的博客，怎么说呢，基于Github建立的博客源码都是托管在Github上面的，我们完全可以找到每篇博客的源码。如果你想看看别人的博客是怎么配置的，可以去看他的博客源码souce分支下的_config.yml文件，如果想看别人的博客用MarkDown怎么写的，可以看source分支source目录下的\_posts路径下的MarkDown文件，这比看各种各样的教程与介绍直观多了，而且很快能找到自己想要的。比如可以参考[唐巧的博客](http://blog.devtang.com/)与对应的[源码托管处](https://github.com/tangqiaoboy/tangqiaoboy.github.com)，看看他的博客怎么配置的、文章是怎么用MarkDown写出来的，然后自己照葫芦画瓢就行了。

编辑md文件我是用Sublime Text2的，供参考。



###发表博文常用命令：
```

#创建一篇博文
rake new_post["post title"] #将在工作目录的source/_post/目录下生成相应的markdown文件。然后可以使用mou工具去修改编辑内容。

#生成预览
rake preview #可以在浏览器中访问localhost:4000在本机实时观察最新的编辑效果。

#在线发布
rake deploy  #这一步将最新的内容push到Github上的master分支，完成部署。成功后，即可在线访问。

#向github提交源文件更新
git add .
git commit -m "提交内容"
git push origin source


```

相对于其它博客，搭建这样的博客还是有点折腾的，费了我不少时间，还好对于我来说不是多大的问题。对于普通用户而言，恐怕就没那么简单了，技术门槛摆在那里是个问题。这种门槛也并非是坏事，至少将一些别有用心的人挡在了Github外面，SourceForge是前车之鉴啊。





