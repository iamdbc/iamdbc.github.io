---
layout: post
comments: true
title:  "PostgreSQL 对大数据量排序时较慢的处理方法"
date:   2019-10-25 22:00:00 +0800
categories: PostgreSQL
---

使用 PostgreSQL 对大数据量排序的时候非常慢，使用 explain 会看到，最耗时的部分是在 sort 的时候，里面用到 external merge disk，这说明，内存已经不够排序了，用到了磁盘空间，这时效率是很低的。这种情况下可以考虑调高 work_mem，使用内存排序，性能会得到一些提升，设置方法如下

{% highlight sql %}
-- 查看当前 work_mem 数值
SHOW work_mem;

-- 设置 work_mem
SET work_mem = '800MB';

-- execute query 执行查询

-- 将 work_mem 重置
RESET work_mem;

{% endhighlight %}

上面的方法中 `SET work_mem = '800MB'` 只针对当前 session 有效，查询结束后将 work_mem 重置。因为调高了 work_mem，当出现大并发时，可能会导致 PG OOM，需要谨慎操作。

参考:
<a href="https://andreigridnev.com/blog/2016-04-16-increase-work_mem-parameter-in-postgresql-to-make-expensive-queries-faster/" target="_blank">https://andreigridnev.com/blog/2016-04-16-increase-work_mem-parameter-in-postgresql-to-make-expensive-queries-faster/</a>

<a href="https://www.cybertec-postgresql.com/en/postgresql-improving-sort-performance/" target="_blank">https://www.cybertec-postgresql.com/en/postgresql-improving-sort-performance/</a>

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
