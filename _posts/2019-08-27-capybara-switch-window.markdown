---
layout: post
comments: true
title:  "Capybara 测试新打开 window"
date:   2019-08-27 20:11:53 +0800
categories: test
---

在测试中，遇到新打开一个 window 的情况，例如 target="_blank"，如果直接检测页面元素，会发现始终检测不到  
这是因为 capybara 并不知道切换 window，这时候要使用

{% highlight ruby %}
Then /^I click on the "([^"]*)" link with new window and see "([^"]*)"$/ do |link, message|
  new_window = window_opened_by { click_link link }

  within_window new_window do
    expect(page).to have_content message
  end
end
{% endhighlight %}


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
