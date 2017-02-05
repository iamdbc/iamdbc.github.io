---
layout: post
title:  "rake 执行时传递参数"
date:   2017-02-05 23:50:00 +0800
categories: articles
keywords: rake arguments zsh
---

有时候执行 `rake` 时需要加参数，代码如下:

```ruby
namespace :user do
  desc 'add new user'
  task :create, [:name, :password] => :environment do |task, args|
    user = User.new(name: args.name, password: args.password)
    if user.save
      p 'success'
    else
      p user.errors.full_messages
    end
  end
end
```

命令执行如下:

```bash
$ rake user:create[name,password]
```

但是在zsh下会提示错误:

```bash
zsh: no matches found: user:create[name,password]
```

其实如果使用zsh的命令自动补全， `rake` 会自动给 `[]` 加 `\` 转义，或者我们给命令加 `'` 也可以:

```bash
$ rake user:create\[name,password\]
$ rake 'user:create[name, password]'
```
