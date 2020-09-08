---
layout: post
comments: true
title:  "Rails 获取 model enum value"
date:   2019-09-09 08:22:00 +0800
categories: ruby-on-rails
---

举例来说，user 表有以下的 enum

{% highlight ruby %}

class User < ApplicationRecord
    enum platform: { mini_program: 1, iOS: 2, Android: 3 }
end

user.platform
=> "iOS"
{% endhighlight %}

以下方式可以获取 iOS 对应的 value

{% highlight ruby %}
User.platforms[user.platform]
=> 1

user.platform_before_type_case
=> 1

user.read_attribute_before_type_cast('platform')
=> 1
{% endhighlight %}

参考: <a href="https://rubyinrails.com/2019/04/12/how-to-get-integer-value-from-enum-in-rails/" target="_blank">https://rubyinrails.com/2019/04/12/how-to-get-integer-value-from-enum-in-rails/</a>


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
