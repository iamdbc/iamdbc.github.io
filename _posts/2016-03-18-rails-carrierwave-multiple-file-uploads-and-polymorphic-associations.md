---
layout: post
title:  "carrierwave上传多个图片以及多态关联"
date:   2016-03-18 01:10:00 +0800
categories: articles
keywords: rails, carrierwave, multiple, file, polymorphic
---
**carrierwave多图上传**
[官方文档](https://github.com/carrierwaveuploader/carrierwave)

model:
{% highlight ruby %}
class User < ActiveRecord::Base
  mount_uploaders :avatars, AvatarUploader
end
{% endhighlight %}

view:
{% highlight ruby %}
<%= form.file_field :avatars, multiple: true %>
{% endhighlight %}

controller:
{% highlight ruby %}
params.require(:user).permit(:email, :first_name, :last_name, {avatars: []})

u = User.new(params[:user])
u.save!
u.avatars[0].url # => '/url/to/file.png'
u.avatars[0].current_path # => 'path/to/file.png'
u.avatars[0].identifier # => 'file.png'
{% endhighlight %}

**rails Active Record的多态关联**
[官方文档](http://guides.ruby-china.org/association_basics.html#%E5%A4%9A%E6%80%81%E5%85%B3%E8%81%94)

一个模型可以同时属于多个模型的方法，例如图片模型可以同时属于员工/产品这两个模型。

{% highlight ruby linenos %}
# model Picture 可以同时属于多个 model
class Picture < ActiveRecord::Base
  belongs_to :imageable, polymorphic: true
end

# model Employee 拥有很多 pictures
class Employee < ActiveRecord::Base
  has_many :pictures, as: :imageable
end

# model Product 拥有很多 pictures
class Product < ActiveRecord::Base
  has_many :pictures, as: :imageable
end
{% endhighlight %}

migration
{% highlight ruby %}
class CreatePictures < ActiveRecord::Migration
  def change
    create_table :pictures do |t|
      t.string :image
      # 会同时生成 imageable_id, imageable_type 这两个字段
      t.references :imageable, polymorphic: true
      t.timestamps
    end
  end
end
{% endhighlight %}

imageable_id 对应的是所属模型的id， imageable_type 对应的是所属模型的名称

使用方法
{% highlight ruby linenos %}
product = Product.find(1)
img_path = 'path/image'
Picture.create(image: img_path, imageable: product)
# 以上将会在pictures表里生成一行记录， image => img_path, imageable_id => product.id, imageable_type => 'product'
product.pictures
{% endhighlight %}
