---
layout: post
title:  "rails model 中使用 enum"
date:   2017-01-07 18:00:00 +0800
categories: articles
keywords: rails model enum where
---

在数据库里，有时我们会使用 integer 表示不同状态，例如 status 字段，1表示正常，2表示不正常。

但是这种数字多起来之后，就会很难管理。

rails model 中的 enum 就是用来处理这种问题的。

{% highlight ruby %}
# product.rb
class Product < ApplicationRecord
  enum status: { valid: 1, invalid: 2 }
end
{% endhighlight %}

我们可以得到如下几种方法:

{% highlight ruby %}
product.valid! # 设置 product status 为 valid

product.valid? # 检查 product status 是否为 valid

product.status # 输出当前 status 对应的字面量 "valid" / "invalid"

product.status = "valid" # 修改 status 为 1

Product.statuses # 输出 {"valid" => 1, "invalid" => 2}

Product.valid # 查询 products 表中 status = 1 的数据
{% endhighlight %}

在使用 enum 查询的时候，rails 不同版本会有不同的结果。

## rails 5 的结果如下
{% highlight ruby %}
Product.where(status: "valid") # 可以直接使用字面量作为查询条件
{% endhighlight %}

## rails 4.2 的结果如下
{% highlight ruby %}
Product.where(status: "valid") # 查询条件始终是 status = 0

# 需要改成下面的形式
Product.where(status: Product.statuses[:valid])
# 或者还是写原始数值
Product.where(status: 1)
{% endhighlight %}


