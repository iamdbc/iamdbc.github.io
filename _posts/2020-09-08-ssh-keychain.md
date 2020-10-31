---
layout: post
title:  "macOS 添加 keychain"
date:   2020-09-08 23:50:00 +0800
categories: macOS
keywords: macOS keychain
comments: true
---

把 SSH key 加入到 macOS 的 keychain，避免每次都要指定 SSH private key 的位置

1. ssh-add -K ~/.ssh/[your-private-key]
2. 配置 .ssh/config

```config
Host hostname
     UseKeychain yes # 增加这行
     HostName you_host_name
     User root
     PreferredAuthentications publickey
     IdentityFile ~/.ssh/key

# 增加下面配置
host *
   ForwardAgent yes
   AddKeysToAgent yes
```

参考:
https://apple.stackexchange.com/questions/48502/how-can-i-permanently-add-my-ssh-private-key-to-keychain-so-it-is-automatically
https://segmentfault.com/q/1010000000835302


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
