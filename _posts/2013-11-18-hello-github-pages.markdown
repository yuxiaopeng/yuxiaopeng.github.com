---
layout: post
title: Hello Github Pages
date: 2013-11-18 11:56
comments: true
tags:
---

![](/assets/images/2013/11/octopress-logo.png)

### 大家好，我是Roc，欢迎大家关注我的小站！

`博客`->`托管博客`->`wordpress`->`octopress`<br>建博后第一篇文章，就说说创建这个Blog的过程吧。最初接触博客这个概念是在2007年左右，那时候网吧很流行，而去网吧能干的事无非是玩游戏、聊QQ、浏览网页，我不是一个对游戏能够痴迷的人，浏览网页的过程中发现了51，后来才知道这不是偶然，51在一线城市可能没多少人用，但三线及以下城市很多人用51。在51你会发现博客中展示更多的不是文章而是照片，用户中多半是年轻女性，这可能是51抓住了年轻女性喜欢展示自己漂亮照片的心理。51让我知道了博客这个概念，后来陆陆续续使用了Blogger、新浪博客、搜狐博客、博客中国、QQ空间等。2010年左右经常逛一些技术论坛，知道了独立博客的概念，区别于托管博客，独立博客有自己的空间、域名，更自由、灵活，不受限制，典型代表是wordpress。于是研究之，中间搭建过几个博客，也尝试自己做过wordpress主题，但这也只是对wordpress的相关技术的了解，并没有利用到它强大的文章功能，这也是不能坚持写博客的原因。2012年认识了Github，经常去Fork一些代码学习，后来知道了Page,发现竟然有如此强大的博客系统,最终在2013年初创建了这个博客，拖到现在才写了这第一篇文章。下面是我创建博客的过程：

### 搭建Octopress

### 1.安装 ruby
下载地址: [`RubyInstaller`](http://rubyforge.org/frs/?group_id=167)，最新版本 rubyinstaller-1.9.3-p429.exe，注意版本是1.9.3。
### 2.安装 gem
下载地址: [`gem`](http://rubyinstaller.org/downloads/)，与1.9.3对应版本为 DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe，解压到D:/DevKit,cmd中执行：


```
cd D:\DevKit
ruby dk.rb init
ruby dk.rb instal
```

### 3.安装 python
下载地址：[`activepython`](http://www.activestate.com/activepython/downloads) ，安装2.7版，博客代码加亮模块需要python环境的支持，安装完以后，在win的cmd窗口中执行：

```
easy_install pygments
```

win7环境变量：<br>
LANG=zh_CN.UTF-8
LC_ALL=zh_CN.UTF-8

更新gem源:<br>

```
gem sources --remove http://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l
```

注意 ：请确保只有 http://ruby.taobao.org/ 唯一一个条目

安装 rdoc 和 bundler:<br>
gem install rdoc bundler
### 4.安装 Octopress

```
git clone git://github.com/imathis/octopress.git octopress
bundle install
rake install
```

### 5.发布到github pages
与github建立连接:

```
rake setup_github_pages
```

按照提示输入 github page repository的url地址，例如：git@github.com:yuxiaopeng/yuxiaopeng.github.com.git

生成静态页面:

```
rake generate
```

本地预览，访问 http://localhost:4000 查看博客本地运行效果:

```
rake preview
```

部署到github 服务器：

```
rake deploy
```

