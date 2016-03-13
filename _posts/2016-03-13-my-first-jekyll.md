---
layout: post
title:  "我的第一篇Jekyll博客"
date:   2016-03-13 20:29:09 +0800
categories: Jekyll rails
---
晚上看了一下[Jekyll](http://jekyllrb.com/)的结构和写法，以后就试着用Jekyll来写博客了。

写点rails部署中我踩的坑吧。其实也没什么，主要是自己总不注意看报错信息，瞎猜问题，才走了不少弯路。

所以说，做开发的，看报错信息很重要！

**1. 记得设置secret_key_base**

页面显示"Incomplete response received from application"。我没看日志，其实日志里都写的有原因。我当时是把passenger的环境设置为development，来看页面报错信息的。不看日志是个坏习惯，尤其是刚部署的时候，没多少记录，基本上很快都能发现问题。

在应用的根目录输入执行下面的命令，复制到./config/secrets.yml,然后重启应用
{% highlight bash %}
$ rake secret
# after copy secret to secrets.yml
$ touch ./tmp/restart.txt
{% endhighlight %}

**2. 关于<%= ENV %>变量**

rails有些配置会使用<%= ENV["PASS"] %>这样的格式，这个ENV在哪设置的呢？

就在bash.rc或者zsh.rc中设置，例如
{% highlight bash %}
export PASS='password'
{% endhighlight %}
