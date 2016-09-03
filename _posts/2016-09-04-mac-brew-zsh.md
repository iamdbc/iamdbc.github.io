---
layout: post
title:  "Mac切换zsh"
date:   2016-09-04 00:25:00 +0800
categories: articles
keywords: Mac brew zsh
---

{% highlight bash %}
$ which zsh
# /usr/bin/zsh 说明是系统自带zsh
$ brew install zsh
# 安装zsh...
$ which zsh
# /usr/local/bin/zsh 可以看到软连接是brew安装的zsh
$ sudo vi /etc/shells
# 把 /usr/local/bin/zsh 写入文件
$ chsh -s /usr/local/bin/zsh
$ which zsh
# /usr/local/bin/zsh
$ echo $SHELL
# /usr/local/bin/zsh
$ zsh --version
# 查看zsh版本
{% endhighlight %}
