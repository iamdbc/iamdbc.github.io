---
layout: post
title:  "在rails中使用grape写api"
date:   2016-12-05 00:00:00 +0800
categories: articles
keywords: rails api grape grape-entity grape-swagger
---

最近学着在rails中使用grape写api。由于官方文档比较简单，这里试着把grape的目录结构化一下，方便开发。

## 0 安装

{% highlight bash %}
# gemfile
gem 'grape'
gem 'grape-entity'
gem 'grape-swagger'
{% endhighlight %}
<br>

## 1 目录结构

{% highlight text %}

api
└── v1
      ├── entities
      │    └── product.rb
      ├── helpers
      │    └── base_helper.rb
      ├── api.rb
      └── products_api.rb
{% endhighlight %}

api.rb 保存api的基本信息，包括版本，格式，并负责加载其它文件
{% highlight ruby %}
# api.rb
require 'grape-swagger'

module V1
  class API < Grape::API
    version 'v1'
    format  :json
    prefix  :api

    helpers V1::Helpers::BaseHelpers

    mount V1::ProductsAPI

    add_swagger_documentation

  end
end
{% endhighlight %}

products_api.rb 相当于controller
{% highlight ruby %}
# products_api.rb
module V1
  class ProductsAPI < V1::API
    resource :products do
      desc 'Returns products'
      get '/' do
        "Products goes here"
      end

      desc 'Returns product by id'
      params do
        requires :id, type: Integer, desc: 'Product id.'
      end
      route_param :id do
        get do
          authenticate!
          product = current_user.products.find_by(id: params[:id])

          if product.present?
            present product, with: V1::Entities::Product
          else
            error!({ errno: 46004, errmsg: 'product not exists' })
          end
        end
      end

    end

  end
end

{% endhighlight %}

entities 目录下定义返回数据的实体，例如entities/product.rb和关联的product_category.rb
{% highlight ruby %}
# entities/product.rb
module V1
  module Entities
    class Product < Grape::Entity
      format_with(:timestamp) { |dt| dt.to_i }

      expose :name
      expose :price
      expose :status
      expose :stock
      expose :sales

      # product_category 是在rails的model中定义的关联，在这里可以直接用
      expose :product_category, using: V1::Entities::ProductCategory

      with_options(format_with: :timestamp) do
        expose :created_at, documentation: { type: 'Timestamp' }
      end
    end
  end
end
{% endhighlight %}

{% highlight ruby %}
# entities/product_category.rb
module V1
  module Entities
    class ProductCategory < Grape::Entity

      expose :name

    end
  end
end

{% endhighlight %}

helpers目录下放公共方法
{% highlight ruby %}
# helpers/base_helper.rb
module V1
  module Helpers
    module BaseHelpers
      extend Grape::API::Helpers

      def api_params
        env['api.endpoint'].params.to_hash
      end

      def current_user
        @current_user ||= User.first
      end

      def authenticate!
        error!({ errcode: 40000, errmsg: 'invalid user' }) unless current_user
      end

    end
  end
end
{% endhighlight %}

routes.rb
{% highlight ruby %}
# routes.rb
mount V1::API => '/'
{% endhighlight %}

调用方式
{% highlight text %}
http://localhost:3000/v1/products
{% endhighlight %}
<br>

## 2 自动生成 api 文档
grape-swagger 是用来自动生成api文档的，已经在 api.rb 中加载了，之后进入下面url就会看到自动生成的文档，只是没有样式，还需要再加入文档的样式表，参考[swagger-ui](https://github.com/swagger-api/swagger-ui)。
{% highlight text %}
http://localhost:3000/api/v1/swagger_doc
{% endhighlight %}
