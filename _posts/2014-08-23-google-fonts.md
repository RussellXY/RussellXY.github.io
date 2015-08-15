---
layout: post
title: 解决jekyll博客加载缓慢的问题
date: 2014-08-23
comments: true
categories: jekyll
tags: 
- jekyll - fonts google
keywords: jekyll fonts google
description: 解决jekyll博客加载缓慢的问题
published: true
---


最近开始用jekyll并通过Github Pags服务来写个人技术博客了，在使用的过程中发现了一个很恶心的问题：不管是直接访问github上自己的博客还是在本地开启jekyll服务器访问博客都非常的慢，没错，连本地打开都非！常！慢！

在一次郁闷的等待本地博客更新来测试自己写的markdown代码效果的时候，偶然的发现在打开博客时浏览器的左下方出现了一行字:"等待fonts.googleapis.com回应"，这下才恍然大悟，原来是在加载博客页面的时候还要去获取一些谷歌的字体资源，大谷歌被xx已经是好多年前的事情了，所以才导致了博客访问速度很慢的问题。

上网一搜发现了一个不错的办法：使用360网站卫士推出的一项字体加速服务，只需要修改一行代码就可以免费使用到由360网站卫士CDN加速的字体服务。

在博客目录下进入`_layouts`文件夹，使用编辑器打开`default.html`文件，搜索`fonts.googleapis.com`，然后将这个url替换成`fonts.useso.com`。

大功告成，打开本地博客和github博客一试，秒开有木有！速度感人有木有！
