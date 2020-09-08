---
layout: post
comments: true
title:  "Swift 前导加 0 的处理"
date:   2019-12-01 22:00:00 +0800
categories: Swift
---

{% highlight swift %}
let hours = 3
let minutes = 15
let seconds = 7
print(String(format: "%02d:%02d:%02d", hours, minutes, seconds))

// 03:15:07
{% endhighlight %}

参考:
<a href="https://stackoverflow.com/questions/25566581/leading-zeros-for-int-in-swift" target="_blank">https://stackoverflow.com/questions/25566581/leading-zeros-for-int-in-swift</a>

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
