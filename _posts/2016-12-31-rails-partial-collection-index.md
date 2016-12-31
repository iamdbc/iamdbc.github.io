---
layout: post
title:  "rails render partial collection 有 index"
date:   2016-12-31 23:10:00 +0800
categories: articles
keywords: rails render partial collection partial
---

例如在view中要遍历@products，代码如下
{% highlight erb %}
<%= render partial: "product", collection: @products %>
{% endhighlight %}

在_product.html.erb中
{% highlight erb %}
<%= product_counter %>
{% endhighlight %}
即可输出当前 product 的索引值了
