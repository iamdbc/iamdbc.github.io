---
layout: post
comments: true
title:  "Rails 导出大容量 csv"
date:   2019-10-08 20:29:00 +0800
categories: ruby-on-rails
---

有时候我们可能需要从数据库查询大量的结果然后导出 csv，如果直接 send_data，可能需要占用大量内存或者导致超时，这时候可以用 stream 方式导出，这样用分页的方式把数据分批输出到浏览器，直到数据传输完毕。同时可以调整 limit 的值，来决定导出的快慢，但是要注意内存占用情况。

```ruby
def export_action
  respond_to do |format|  
    format.csv do
      filename = "large_file.csv"

      headers.delete("Content-Length")
      headers["Cache-Control"] = "no-cache"
      headers["Content-Type"] = "text/csv"
      headers["Content-Disposition"] = "attachment; filename=\"#{filename}\""
      headers["X-Accel-Buffering"] = "no"

      self.response_body = export_csv

      response.status = 200
    end
  end
end

def export_csv
  Enumerator.new do |yielder|
    # header: your csv header
    yielder << CSV.generate_line(header)

    # iterate your data to export
    limit = 5000
    pages = (total_count.to_i / limit.to_f).ceil

    pages.times do |offset_idx|
      Model.limit(limit).offset(offset_idx * limit).each do |row_result|
        yielder << CSV.generate_line(cvs_report_row(row_result))
      end
    end
  end
end
      
# format your csv row
def csv_report_row(row_result)
  csv_row = []
  csv_row << row_result.column
  csv_row
end
```


参考:

<a href="https://medium.com/table-xi/stream-csv-files-in-rails-because-you-can-46c212159ab7" target="_blank">https://medium.com/table-xi/stream-csv-files-in-rails-because-you-can-46c212159ab7</a>

<a href="https://coderwall.com/p/kad56a/streaming-large-data-responses-with-rails
" target="_blank">https://coderwall.com/p/kad56a/streaming-large-data-responses-with-rails
</a>


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
