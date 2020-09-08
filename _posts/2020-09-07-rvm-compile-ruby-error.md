---
layout: post
title:  "因为 xcode-beta 的原因导致 rvm 安装 ruby 时编译报错"
date:   2020-09-07 23:50:00 +0800
categories: ruby rvm
keywords: rvm ruby compile gcc xcode-beta
comments: true
---

最近闲来无事，把一些年久失修的软件升级了一下，结果就导致今天需要重新安装一个 ruby 版本时无法通过编译

首先查看 make.log，提示找不到 `/Library/Developer/CommandLineTools/SDKs`，这个可以通过 `xcode-select --install` 重新安装来解决

之后再次运行 `rvm install` 仍然报错，继续查看 make.log

log 有 6 万多行，看到有很多 warning，这些应该不是导致编译中断的原因，直接搜索关键词 `error`，
查到是 `mjit_compile.c` 编译报错

再搜索一下相关报错，没有找到问题的原因，但是通过其他关键词猜到可能和 gcc version 有关

在 terminal 输入 `gcc --version` 可以看到 `--with-gxx-include-dir` 的路径为 xcode 的 SDKs 目录
突然想到本地安装了 xcode-beta，是否是和这个相关？
于是进入 xcode-beta，在 Preferences 里找到 Locations，
把 Command Line Tools 的 path 修改为 xcode-beta 的路径(之前是 xcode 正式版路径)

再次执行 `rvm install`

这时又出现一些软件版本报错，可以通过参数把相关软件版本指向 brew 的路径，例如我遇到的问题是通过下面方式来解决的

```bash
rvm install 2.6.5 \
  --disable-binary \
  --with-opt-dir=$(brew --prefix readline)
```

{% if page.comments %}
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://iamdbc-eggy.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
                            
{% endif %}
