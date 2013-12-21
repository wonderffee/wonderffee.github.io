---
layout: post
title: "update CocoaPods问题小记"
date: 2013-12-21 21:51
comments: true
categories: tools
---

执行pod install时出错，提示如下：
```bash
The version of CocoaPods used to generate the lockfile is higher that the one of the current executable. Incompatibility issues might arise.
```

问题原因：CocoaPods版本低了

解决办法：执行gem update cocoapod更新CocoaPods

可以通过gem list来查看本机上所有已经安装的gem包

<!--more-->
粗心执行了gem update，导致更新了全部的gem包，执行rake new_post["title"]带来了新的问题：
```bash
rake aborted!
You have already activated rake 10.1.1, but your Gemfile requires rake 0.9.2.2. Using bundle exec may solve this.
```

问题原因：gem update更新了rake到10.1.1，而Octopress的Gemfile中要求的rake版本为0.9

解决方法：修改Octopress的Gemfile，将gem 'rake', '~> 0.9'改为gem 'rake', '~> 10.1.1'，重新执行rake new_post["title"]就没有问题了。
