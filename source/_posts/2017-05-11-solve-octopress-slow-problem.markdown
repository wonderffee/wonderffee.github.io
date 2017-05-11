---
layout: post
title: "解决Octopress博客访问太慢的问题"
date: 2017-05-11 14:54
comments: true
categories: octopress
---
博客好久没动了，也一直有一个问题没解决，就是打开博客太慢了。

这得从Octopress引用了Google的字体文件和jQuery文件说起，刚开始建博客就发现了这个问题。当时也解决了，使用了360推出的常用前端公共库CDN服务libs.useso.com，岂料后来360也关闭了这项服务，访问再次变慢，然后就一直没管。

今天就来解决一下吧，综合各种搜索资料，有了以下的集大成版：
<!--more-->
##1.加速jQuery访问
办法就是把jquery的cdn服务改成microsoft的。

编辑文件 source/_includes/head.html 找到下图注释掉的部分,替换成

```javascript
<!--<script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>-->
  <script src="//ajax.aspnetcdn.com/ajax/jQuery/jquery-1.9.1.min.js"></script>
```

##2.加速font访问

办法就是把font改成使用本地的。

新建一个目录 source/fonts，后面下载的字体就放在这个fonts文件夹中。访问[PT_Sans](https://github.com/google/fonts/tree/master/ofl/ptsans)和[PT_Serif](https://github.com/google/fonts/tree/master/ofl/ptserif)下载两字体对应的字体文件，分别存放于PT_Sans和PT_Serif文件夹中，各4个ttf字体。

这里有一个问题，给的链接都是Github中repo库的子文件夹链接，怎么一次性下载子文件夹呢？还真有办法，打开[gitzip](http://kinolien.github.io/gitzip/),在右上角文本框中粘贴子文件夹网址点Download就可以下载了，so easy!

如果不下载子文件夹，也可以安装[OctoTree](https://chrome.google.com/webstore/detail/octotree/bkhaagjahfmjljalopjnoealnfndnagc)这个Chrome插件一个一个安装，就是稍微麻烦一些。

下载后的字体存放目录层级如下：

```
.
├── PT_Sans
│   ├── OFL.txt
│   ├── PT_Sans-Web-BoldItalic.ttf
│   ├── PT_Sans-Web-Bold.ttf
│   ├── PT_Sans-Web-Italic.ttf
│   └── PT_Sans-Web-Regular.ttf
└── PT_Serif
    ├── OFL.txt
    ├── PT_Serif-Web-BoldItalic.ttf
    ├── PT_Serif-Web-Bold.ttf
    ├── PT_Serif-Web-Italic.ttf
    └── PT_Serif-Web-Regular.ttf
```

接着在 source/stylesheets 下新建pt_sans.css和pt_serif.css替代原先的google fonts代码。这两个文件的内容分别是：

{% codeblock pt_sans.css lang:css %}
@font-face {
  font-family: 'PT Sans';
  font-style: normal;
  font-weight: 400;
  src: local('PT Sans'), local('PTSans-Regular'), url(/fonts/PT_Sans/PT_Sans-Web-Regular.ttf) format('truetype');
}
@font-face {
  font-family: 'PT Sans';
  font-style: normal;
  font-weight: 700;
  src: local('PT Sans Bold'), local('PTSans-Bold'), url(/fonts/PT_Sans/PT_Sans-Web-Bold.ttf) format('truetype');
}
@font-face {
  font-family: 'PT Sans';
  font-style: italic;
  font-weight: 400;
  src: local('PT Sans Italic'), local('PTSans-Italic'), url(/fonts/PT_Sans/PT_Sans-Web-Italic.ttf) format('truetype');
}
@font-face {
  font-family: 'PT Sans';
  font-style: italic;
  font-weight: 700;
  src: local('PT Sans Bold Italic'), local('PTSans-BoldItalic'), url(/fonts/PT_Sans/PT_Sans-Web-BoldItalic.ttf) format('truetype');
}
{% endcodeblock %}


{% codeblock pt_serif.css lang:css %}
@font-face {
  font-family: 'PT Serif';
  font-style: normal;
  font-weight: 400;
  src: local('PT Serif'), local('PTSerif-Regular'), url(/fonts/PT_Serif/PT_Serif-Web-Regular.ttf) format('truetype');
}
@font-face {
  font-family: 'PT Serif';
  font-style: normal;
  font-weight: 700;
  src: local('PT Serif Bold'), local('PTSerif-Bold'), url(/fonts/PT_Serif/PT_Serif-Web-Bold.ttf) format('truetype');
}
@font-face {
  font-family: 'PT Serif';
  font-style: italic;
  font-weight: 400;
  src: local('PT Serif Italic'), local('PTSerif-Italic'), url(/fonts/PT_Serif/PT_Serif-Web-Italic.ttf) format('truetype');
}
@font-face {
  font-family: 'PT Serif';
  font-style: italic;
  font-weight: 700;
  src: local('PT Serif Bold Italic'), local('PTSerif-BoldItalic'), url(/fonts/PT_Serif/PT_Serif-Web-BoldItalic.ttf) format('truetype');
}
{% endcodeblock %}

然后修改source/_includes/custom/head.html

```html
<!--Fonts from Google"s Web font directory at http://google.com/webfonts -->
<link href="/stylesheets/pt_serif.css" rel="stylesheet" type="text/css">
<link href="/stylesheets/pt_sans.css" rel="stylesheet" type="text/css">
```

##3.重新部署

重新部署自然就简单了，可以用preview先预览效果:

```
rake generate
rake preview
rake deploy
```

##参考
* [加快Octopress国内访问速度 - 冷雨之家](http://douxinchun.github.io/blog/20150511/accelerate-the-speed-of-access-octopress-in-china.html)
* [加快octopress国内访问速度 - 颇忒脱’s Blog](http://www.chanjar.me/blog/2014/06/29/jia-kuai-octopressguo-nei-fang-wen-su-du)
* [个性化我们的Octopress博客 - 简书](http://www.jianshu.com/p/fe0e089a985c)
* [记录：Octopress配置修改之旅 - Jacky and MSC](https://old.jacky-sj.com/blog/2014/02/26/record2/)
* [Octopress加速Google字体渲染 - IMXYLZ](http://imxylz.com/blog/2013/09/22/move-google-fonts-to-local-server/)
* [Octopress 加速 - GeekGaoyang’s Blog](http://geekgaoyang.herokuapp.com/blog/2015/02/28/octopress-speed-up/)