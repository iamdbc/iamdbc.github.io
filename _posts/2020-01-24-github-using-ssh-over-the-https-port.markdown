---
layout: post
comments: true
title:  "GitHub ssh port 22 端口访问超时的解决办法"
date:   2020-01-24 20:00:00 +0800
categories: experience
---

趁着过年这两天，想着把 Mojave 升级到 Catalina

升级过程倒是还算顺利，但是升级完之后想从 GitHub pull 代码时，却一直提示下面的错误

`ssh: connect to host github.com port 22: Operation timed out`

之后我尝试`ssh -T -p 443 git@github.com`倒是提示正常

这说明443端口是可以用的，不行就切换到https，先不用ssh了

但是我切换到https后发现还是没办法pull代码

后面我去查了GitHub的文档，发现可以改为使用https端口来使用ssh，先用下面命令测试一下https端口是否可以正常使用

{% highlight sql %}
$ ssh -T -p 443 git@ssh.github.com
> Hi username! You've successfully authenticated, but GitHub does not
> provide shell access.
{% endhighlight %}

之后在`~/.ssh/config`中修改以下的配置

```bash
Host github.com
  Hostname ssh.github.com
  Port 443
```

最后再测试一下，如果像下面那样提示，就成功了，可以正常pull代码了
```bash
$ ssh -T git@github.com
> Hi username! You've successfully authenticated, but GitHub does not
> provide shell access.
```

参考:
<a href="https://help.github.com/en/github/authenticating-to-github/using-ssh-over-the-https-port" target="_blank">https://help.github.com/en/github/authenticating-to-github/using-ssh-over-the-https-port</a>

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
