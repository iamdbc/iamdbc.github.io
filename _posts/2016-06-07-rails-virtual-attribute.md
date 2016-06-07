---
layout: post
title:  "Rails 虚拟属性"
date:   2016-06-07 22:00:00 +0800
categories: articles
keywords: rails 虚拟属性 PHP
---

虚拟属性就是数据表中不存在的属性。

像PHP，在提交表单的时候，要想让提交的字段直接保存到数据库，就要保证数据库里有对应的字段，例如
{% highlight html %}
<input type="password" name="password" />
{% endhighlight %}

后端接收password，然后计算salt，将password和salt组合获取password_digest再存入数据库。

因此针对password，就没办法和表单中其他数据库已经存在的属性一起，放在数组里直接插入数据库，因为数据库里没有password字段。

在Rails中，可以在model里使用虚拟属性，例如在user.rb中
{% highlight ruby %}
def password
  @password
end

def password=(password)
  @password = password
  generate_password(password)
end

def generate_password(password)
  salt = Array.new(10) { rand(1024).to_s(36) }.join
  self.salt, self.password_digest =
      salt, Digest::SHA256.hexdigest(pass + salt)
end
{% endhighlight %}

使用user.password = 'newpassword'

调用user.password会返回用户的password，即'newpassword'

在提交保存或者更新password的时候，会调用password=这个方法，将model的真实属性salt和password_digest进行赋值。

这样的话，就可以把password直接做为提交参数，对model进行操作了。
