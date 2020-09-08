---
layout: post
comments: true
title:  "Ruby 参数关键字 **"
date:   2019-09-12 08:29:00 +0800
categories: ruby
---

如果不知道后面可能用到什么参数，最后一个参数可以直接写成 `**`(Double **Splat)，它会接受所有 hash 类型的参数

{% highlight ruby %}
def test(a, b, **others)
    p a, b, others, others[:test], others[:no_key]
end

test(1, 2, test: 'test string', test_another: 'test another string')

=> 
1
2
{:test=>"test string", :test_another=>"test another string"}
"test string"
nil
{% endhighlight %}

参考:

<a href="https://flushentitypacket.github.io/ruby/2015/03/31/ruby-keyword-arguments-the-double-splat-and-starsnake.html" target="_blank">https://flushentitypacket.github.io/ruby/2015/03/31/ruby-keyword-arguments-the-double-splat-and-starsnake.html</a>

<a href="https://www.rubyguides.com/2018/06/rubys-method-arguments/" target="_blank">https://www.rubyguides.com/2018/06/rubys-method-arguments/</a>

<a href="https://dev.firmafon.dk/blog/drat-ruby-has-a-double-splat/">https://dev.firmafon.dk/blog/drat-ruby-has-a-double-splat/</a>


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
