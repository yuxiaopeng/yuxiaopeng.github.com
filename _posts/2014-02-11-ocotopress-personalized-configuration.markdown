---
layout: post
title: Ocotopress个性化配置
date: 2014-02-11 14:51
tags:
---

![](/assets/images/2014/02/octopress.png)

### 1、添加个人域名

octopress目录下，执行命令：

```
echo 'yourdomain.com' >> source/CNAME
```

此命令会在octopress的source目录创建一个名为CNAME的文件文件，内容为`yourdomain.com`，完成之后，执行如下命令将CNAME部署到github：

```
rake generate
git add .
git commit -am "add CNAME." 
git push origin source
rake deploy
```

然后在你的DNS服务商，如 dnspod.cn，添加相应的CNAME指向 `yourname.github.com`。如果你要使用顶级域名，如 [`http://yuxiaopeng.com`](http://yuxiaopeng.com) 访问你的博客，则需要使用A记录指向 `204.232.175.78`。

详细内容请参考：[`Custom Domains`](http://octopress.org/docs/deploying/github/)


### 2、安装主题

本博使用shashank的greyshade主题，在终端中执行如下命令：

```
$ git clone git@github.com:shashankmehta/greyshade.git .themes/greyshade
$ echo "\$greyshade: color;" >> sass/custom/_colors.scss //Substitue 'color' with your highlight color
$ rake "install[greyshade]"
$ rake generate
```
然后，部署到github：

```
rake generate
git add .
git commit -am "install greyshade." 
git push origin source
rake deploy
```

详细内容请参考：[`greyshade`](https://github.com/shashankmehta/greyshade)


### 3、添加多说

* 获取short_name

去 [`多说网`](http://duoshuo.com) 注册账号，并获取站点的short_name

* 在_config.yml文件中添加如下内容

```
# duoshuo comments
duoshuo_comments: true
duoshuo_short_name: yourname
```

* 在source/_layouts/post.html中添加多说评论模块

```
｛% if site.duoshuo_short_name and site.duoshuo_comments == true and page.comments == true %｝
  <section>
    <h1>Comments</h1>
    <div id="comments" aria-live="polite">｛% include post/duoshuo.html %｝</div>
  </section>
｛% endif %｝
```

* 创建source/_includes/post/duoshuo.html，并填入如下内容

```
<!-- Duoshuo Comment BEGIN -->
<div class="ds-thread"></div>
<script type="text/javascript">
  var duoshuoQuery = {short_name:"{{ site.duoshuo_short_name }}"};
  (function() {
    var ds = document.createElement('script');
    ds.type = 'text/javascript';ds.async = true;
    ds.src = 'http://static.duoshuo.com/embed.js';
    ds.charset = 'UTF-8';
    (document.getElementsByTagName('head')[0] 
    || document.getElementsByTagName('body')[0]).appendChild(ds);
  })();
</script>
<!-- Duoshuo Comment END -->
```

* 发布到站点

```
$ rake generate
$ git add .
$ git commit -am "添加多说评论" 
$ git push origin source
$ rake deploy
```

### 4、添加百度和Google统计

从百度统计获取脚本，然后添加到文件`source/_includes/after_footer.html`文件中
从google analytics获取跟踪ID，然后将这个ID添加到`_config.yml`文件的`google_analytics_tracking_id`后面即可。

```
# Google Analytics
google_analytics_tracking_id: UA-457s202x-x
```

持续更新

