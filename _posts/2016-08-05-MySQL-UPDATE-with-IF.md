---
layout: post
title:  "MySQL包含IF的UPDATE"
date:   2016-06-07 22:00:00 +0800
categories: articles
keywords: MySQL UPDATE IF ADD_DATE
---

{% highlight sql %}
UPDATE table SET a_datetime =
IF(DATE_FORMAT(b_datetime, '%H') > 18,
DATE_ADD(DATE(b_datetime), INTERVAL '2 20' DAY_HOUR),
DATE_ADD(DATE(b_datetime), INTERVAL '1 20' DAY_HOUR))
{% endhighlight %}

场景描述:

1. 需要更新table内的a_datetime字段
2. a_datetime字段需要根据b字段时间来确认
3. 当b_datetime的hour > 18 的时候，将a_datetime设置为b_datetime后1天的20:00, 否则，将a_datetime设置为b_datetime后2天的20:00

IF:
{% highlight sql %}
IF(condition, condition is true, condition if false)
{% endhighlight %}

DATE_FORMAT:
{% highlight sql %}
--格式化日期， 代码里只取了小时的值
DATE_FORMAT(b_datetime, '%H')
{% endhighlight %}


DATE_ADD:
{% highlight sql %}
--DATE取当前值的日期，即取了b_datetime的0时
--DATE_ADD将第一个参数的日期增加了第二个参数对应的DAY_HOUR值
DATE_ADD(DATE(b_datetime), INTERVAL '2 20' DAY_HOUR)
{% endhighlight %}
