---
layout: post
title: 使用Octopress搭建博客
date: 2014-02-10 16:17
tags:
---

![](/assets/images/2014/02/github_page_and-octopress.png)


### 1、安装 Git.

Mac系统默认已安装[`Git`](http://git-scm.com).在终端输入git --version，应该可以看到电脑中的git版本


### 2、安装 Ruby

Octopress需要Ruby环境，RVM(Ruby Version Manager)负责安装和管理Ruby的环境。我们先在Terminal中输入如下命令，来安装RVM：

```
curl -L https://get.rvm.io | bash -s stable —ruby
```

接着安装Ruby 1.9.3，在终端依次运行如下命令：

```
rvm install 1.9.3
rvm use 1.9.3
rvm ruby gems latest
```

完成上面的操作之后，运行ruby --version应该可以看到ruby 1.9.3环境已经安装好了。

参考  [`Installing Ruby With RVM`](http://octopress.org/docs/setup/rvm/)


###3、安装 Octopress

使用用git命令将octopress从github上clone到本机，如下命令： 

```
git clone git://github.com/imathis/octopress.git octopress
cd octopress    # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).
ruby --version  # Should report Ruby 1.9.3
```

安装相关依赖项：

```
gem install bundler
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
bundle install
```

最后安装默认的Octopress 主题。

```
 rake install
```

参考：[`Octopress Setup`](http://octopress.org/docs/setup/)


### 4、部署到Github Pages

创建一个新的Github仓库，并以username.github.com命名 ，其中username是你的GitHub上的用户名或组织名称命名的存储库。 

为了通过url username.github.com访问到我们github pages页面，需要将源码文件提交到Source分支，将生成好的内容提交到Master分支。 Octopress有一个配置可以实现这些。 在终端输入如下命令：
rake setup_github_pages

此命令会要求你输入仓库的url，根据提示，进行输入即可。这个命令将实现如下操作：

1、询问和存储您的Github Pages仓库的URL

2、重命名远程仓库imathis/octopress 的 `origin` 为 `octopress`

3、将Github Pages仓库设置为默认的远程origin分支

4、将活动分支由master切换为source

5、根据存储库配置你的博客的url

6、创建一个_deploy目录，用来存放部署到master分支的内容


接下来执行命令：

```
rake generate
rake deploy
```

上面的命令首先生成博客文件，并将生成的博客文件拷贝到_deploy/目录下，然后将这些内容添加到git中，并commit和push到仓库的master分支。

现在可以访问 http://username.github.com 了。注意：有时候可能会有延时，要等几分钟才能打开。
至此，我们的博客已经完成基本的部署。

不要忘记提交source目录,执行如下命令就可以将source提交到仓库的source分支下。

```
git add .
git commit -m 'your message'
git push origin source
```

参考：[`Deploying to Github Pages`](http://octopress.org/docs/deploying/github/)


### 5、配置 Octopress

Octopress大多数情况下只需要配置`_config.yml`和`Rakefile`文件即可。其中Rakefile是跟博客部署相关，一般情况下并不需要修改这个文件。

config.yml是博客重要的一个配置文件，在config.yml文件中有三大配置项：`Main Configs`、`Jekyll & Plugins`和`3rd Party Settings`。

一般，该文件中其中`url`是必须要填写的，这里的url是在github上创建的仓库地址。另外再修改一下`title`,`subtitle`,`author`，根据需求，开启一些第三方组件服务。

注意格式: 配置项:＋空格＋参数，空格一定不能少，否则会报错。

* 配置日期格式

```
date_format: "%F %a"  		#2014-01-01    
date_format: "%Y年%m月%d日"	#2014年1月1日
```

* Jekyll配置

```
excerpt_link: "继续阅读 &amp;rarr; "
permalink: /blog/:title.html/
```

* 侧边栏

```
default_asides: [asides/recent_posts.html, asides/github.html]
```

* 使用本地时区

将Rakefile里面所有Time.now改为Time.now.localtime

* 自动打开文件/浏览器（编辑Rakefile）


rake new_post/page 后使用指定编辑器自动打开生成的文件 

```
# Misc Configs 中添加，如果不需要自动打开，则直接使用""
editor=\"open\"
# open ,使用系统默认编辑器
# open -a Mou，使用Mou打开
# open -a Byword，使用Byword打开
# subl, 使用Sublime Text2打开 
...
# task :new_post 和 task :new_page 中添加
if #{editor}
	system "sleep 1; #{editor} #{filename}"
end
...
```

rake preview 自动打开浏览器

```
# task :preview 一段中添加
system "sleep 2; open http://localhost:#{server_port}/"
```

参考：[`Configuring Octopress`](http://octopress.org/docs/configuring/)


### 6、开始使用Octopress写博客

博客系统搭建好之后，如何撰写博文呢？通过之前的步骤想必你对octopress的精髓有所了解，没有所见即所得的编辑器让你撰写博文，你要做的是使用 rake new_post 命令创建一篇新的文章，然后使用称手的markdown编辑器进行编辑即可。可选择的markdown编辑器很多，vim，sublime text 2，textmate 2，mou等等。我个人喜欢在osx下使用sublime text 2。

终端中输入命令：

```
rake new_post["title"]
```

Octopress博文存储在source/_posts目录下，按照Jekyll的命名规范对文章进行命名：YYYY-MM-DD-post-title.markdown。文章的名字会被当做url的一部分，而其中的日期用于对博文的区分和排序。打开这个文件内容如下：

```
---
layout: post
title: "使用Octopress搭建博客"
date: 2014-02-10 16:17
comments: true
categories: 
---
```

接着我们就可以在这个文件中写我们的博文啦。完成之后，我们可以预览和部署博文。下面是创建并部署博文的一个完整过程：

```
rake new_post["New Post"]
rake generate
rake preview
git add .
git commit -am "Some comment here." 
git push origin source
rake deploy
```

参考：[`Blogging Basics`](http://octopress.org/docs/blogging/)

