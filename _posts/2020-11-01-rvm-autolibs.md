---
layout: post
title:  "安装 ruby 遇到编译不过的情况 - 使用 rvm autolibs"
date:   2020-11-01 12:50:00 +0800
categories: ruby
keywords: ruby compile rvm
comments: true
---

今天安装 ruby 2.7.2 时又遇到编译不过的情况，重新安装了 CommandLineTools 也无法通过编译

最后是通过设置 rvm autolibs 来解决的
https://rvm.io/rvm/autolibs

```bash
$ rvm autolibs homebrew
```

设置之后，安装 ruby 时会通过 homebrew 安装和更新很多依赖，问题临时解决了，但是也要慎重使用，说不定哪个更新就导致其他软件出现兼容性问题了。
