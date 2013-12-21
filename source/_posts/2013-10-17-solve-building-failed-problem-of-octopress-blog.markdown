---
layout: post
title: "解决Octopress博客Build Failed问题"
date: 2013-10-17 21:11
comments: true
categories: Octopress
---
用了一段时间的Octopress博客，感觉良好，可这毕竟是for程序员的博客，折腾免不了，这不就遇到了一次Build Failed的问题: 执行rake preview命令预览时出现了下面的提示：
<!--more-->
{% codeblock lang:text %}
regeneration: 1 files changed
Liquid Exception: undefined method `[]' for nil:NilClass in atom.xml
/Users/admin/iOSCode/codeGitHub/octopress/plugins/pygments_code.rb:14:in `highlight'
/Users/admin/iOSCode/codeGitHub/octopress/plugins/code_block.rb:82:in `render'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/liquid-2.3.0/lib/liquid/block.rb:94:in `block in render_all'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `collect'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `render_all'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/liquid-2.3.0/lib/liquid/block.rb:82:in `render'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/liquid-2.3.0/lib/liquid/template.rb:124:in `render'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/liquid-2.3.0/lib/liquid/template.rb:132:in `render!'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/jekyll-0.12.0/lib/jekyll/convertible.rb:79:in `do_layout'
/Users/admin/iOSCode/codeGitHub/octopress/plugins/post_filters.rb:167:in `do_layout'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/jekyll-0.12.0/lib/jekyll/page.rb:100:in `render'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/jekyll-0.12.0/lib/jekyll/site.rb:204:in `block in render'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/jekyll-0.12.0/lib/jekyll/site.rb:203:in `each'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/jekyll-0.12.0/lib/jekyll/site.rb:203:in `render'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/jekyll-0.12.0/lib/jekyll/site.rb:41:in `process'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/jekyll-0.12.0/bin/jekyll:253:in `block in <top (required)>'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher.rb:580:in `call'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher.rb:580:in `block in notify_observers'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher.rb:579:in `each'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher.rb:579:in `notify_observers'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher.rb:334:in `block in initialize'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher/scanner.rb:224:in `call'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher/scanner.rb:224:in `notify'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher/scanner.rb:102:in `run_once'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher/scanner.rb:150:in `run_loop'
/Users/admin/.rvm/gems/ruby-1.9.3-p448/gems/directory_watcher-1.4.1/lib/directory_watcher/scanner.rb:45:in `block in start'
Build Failed
{% endcodeblock %}

好大一坨文字，看着就头疼，尝试rake generate也是这样的提示。。这段提示看上去像是ruby的出错堆栈信息，自己不懂ruby， 自然不知如何下手，习惯性地在google上搜索了一下，没找到答案，只好自力更生了。

堆栈信息的第一行说明了问题出现在pygments_code.rb文件中，搜索了一下pygment发现是基于python的代码高亮插件，那就说明代码高亮出了问题，最近更新的一篇文章刚好使用了代码高亮，便有思路了。

先用排除法，找到这篇文章，把其中的代码去掉，再执行rake preview预览，发现没有出现build failed问题，基本可以确定是代码导致的build failed问题。继续用排除法找到导致问题的代码，发现代码行的末尾出现了中文分号+空格，莫非是中文符号的问题？把中文分号+空格替换成英文分号，重试，问题解决！