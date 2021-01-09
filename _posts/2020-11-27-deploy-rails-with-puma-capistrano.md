---
layout: post
title:  "在 ubuntu 20.04 部署 rails production"
date:   2020-11-27 22:00:00 +0800
categories: rails deployment
keywords: rails ruby deployment ubuntu puma capistrano
comments: true
---

因为一台腾讯云服务器最近到期了，所以想借着重新部署的机会，把部署 Rails 的整个流程写成文档，同时把 Rails 和 Ruby 版本也进行升级，

这次部署，同时把老项目的 mina 部署改为了 capistrano 部署。

如果进入服务器发现没有 `.bashrc`，从 `/etc/skel/.bashrc` 复制出来一个

以下操作都是在默认用户下安装，也可以自己创建一个 deploy 用户

---

### 安装 ruby 环境

```bash
# rbevn
$ curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer | bash

$ rbenv init

$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
$ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc

# ruby 和 gem 替换国内镜像源
# https://github.com/andorchen/rbenv-china-mirror
$ export RUBY_BUILD_MIRROR_URL=https://cache.ruby-china.com

$ rbenv install 2.7.2

# update gem
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem sources -l
https://gems.ruby-china.com
# make sure only display gems.ruby-china.com

# install bundler
gem install bundler
```

### Nignx & SSL

```bash
$ sudo apt install nginx
```

在腾讯云可以申请免费的 SSL 证书，之后上传到 `/etc/nginx/conf.d/ssl`

在 `/etc/nginx/site-available` 新增对应 app 的 conf 配置文件，并软连接到 `/etc/nginx/site-enabled`

```conf
# /etc/nginx/site-available/app.conf
upstream myapp {
  # your app puma.socket file
  server unix:///home/deploy/app/your-app/shared/tmp/sockets/puma.sock;
}

server {
  listen 80;
  server_name your-domain.com;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;

  server_name your-domain.com;
  root /home/deploy/app/your-app/current/public;
  index  index.html index.htm;

  access_log /home/deploy/app/your-app/shared/log/nginx.access.log;
  error_log /home/deploy/app/your-app/shared/log/nginx.error.log warn; # only record warn or higher level log

  # tencent cloud ssl cerification files location
  ssl_certificate /etc/nginx/conf.d/ssl/1_your-app_bundle.crt;
  ssl_certificate_key /etc/nginx/conf.d/ssl/2_your-app.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
  ssl_prefer_server_ciphers on;

  location / {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;

    # If the file exists as a static file serve it directly without
    # running all the other rewrite tests on it
    if (-f $request_filename) {
      break;
    }

    # check for index.html for directory index
    # if it's there on the filesystem then rewrite
    # the url to add /index.html to the end of it
    # and then break to send it to the next config rules.
    if (-f $request_filename/index.html) {
      rewrite (.*) $1/index.html break;
    }

    # this is the meat of the rack page caching config
    # it adds .html to the end of the url and then checks
    # the filesystem for that file. If it exists, then we
    # rewrite the url to have explicit .html on the end
    # and then send it on its way to the next config rule.
    # if there is no file on the fs then it sets all the
    # necessary headers and proxies to our upstream pumas
    if (-f $request_filename.html) {
      rewrite (.*) $1.html break;
    }

    if (!-f $request_filename) {
      proxy_pass http://myapp;
      break;
    }
  }

  location ~* \.(ico|css|gif|jpe?g|png|js)(\?[0-9]+)?$ {
     expires max;
     break;
  }

  # Error pages
  # error_page 500 502 503 504 /500.html;
  location = /500.html {
    root /home/deploy/app/your-app/current/public;
  }

}
```

检查 nginx 配置并重新加载配置
```bash
# check configurations is ok
$ sudo nginx -t

$ sudo service nginx reload
```

### Deployment

安装依赖

```bash
# mysql
$ sudo apt install -y build-essential
$ sudo apt install libmysqlclient-dev

# use nvm to install node
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.0/install.sh | bash
$ nvm -v
$ nvm install 12
$ node -v
$ nvm use default 12
$ nvm alias default node
# because execjs can't find nvm node，need to create a soft link, or add alias node=nodejs in .bashrc
# https://github.com/rails/execjs/issues/81
$ sudo ln -s $(which node) /usr/bin/node

# install yarn
# https://classic.yarnpkg.com/en/docs/install#debian-stable
$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
$ sudo apt update && sudo apt install --no-install-recommends yarn
```

使用 systemd 管理 puma
https://github.com/puma/puma/blob/master/docs/systemd.md

如果使用 socket activation 方式来启动 puma，可以在 user level 创建 systemd，否则需要在 root level sudo 启动。因为我没有成功配置 socket activation，所以下面还是使用 sudo 来启动 puma

创建 service 文件 `/etc/systemd/system/puma.service`

```conf
[Unit]
Description=Puma HTTP Server
After=network.target

# Uncomment for socket activation (see below)
# Requires=puma.socket

[Service]
Type=simple

# always failed with watchdog, doesn't know why
# Type=notify
# WatchdogSec=15

Environment=RAILS_ENV=production

WorkingDirectory=/home/deploy/app/your-app/current

# your puma production configuration file location
ExecStart=/home/deploy/.rbenv/shims/bundle exec puma -C /home/deploy/app/your-app/shared/config/puma/production.rb

ExecReload=/home/deploy/.rbenv/shims/bundle exec pumactl -S /home/deploy/app/your-app/shared/tmp/pids/puma.state restart

ExecStop=/home/deploy/.rbenv/shims/bundle exec pumactl -S /home/deploy/app/your-app/shared/tmp/pids/puma.state stop

Restart=always

[Install]
WantedBy=multi-user.target
```

之后可以使用以下命令操作 puma

```bash
$ sudo systemctl daemon-reload

# enable
$ sudo systemctl enable puma.service

# disable
$ sudo systemctl disable puma.service

# start / restart / stop / status
$ sudo systemctl start puma.service
```

puma_production.rb sample

```ruby
rails_env = 'production'
environment rails_env

application = "you-app"

app_path = "/home/deploy/app/#{application}"
shared_path = "#{app_path}/shared"
current_path = "#{app_path}/current"

directory "#{current_path}"


pidfile "#{shared_path}/tmp/pids/puma.pid"
state_path "#{shared_path}/tmp/pids/puma.state"

stdout_redirect "#{shared_path}/log/puma_access.log", "#{shared_path}/log/puma_error.log", true

bind "unix://#{shared_path}/tmp/sockets/puma.sock"

threads_count = ENV.fetch('RAILS_MAX_THREADS') { 5 }
threads threads_count, threads_count
workers 2

port ENV.fetch("PORT") { 3000 }
```

capistrano deploy.rb sample
```ruby
# config valid for current version and patch releases of Capistrano
lock "~> 3.14.1"

set :application, "your-app"
set :repo_url, "your-git-repo-url"

ask :branch, `git rev-parse --abbrev-ref HEAD`.chomp

set :deploy_to, "/home/deploy/app/your-app"

# Depending on your own configuration
append :linked_files, "config/database.yml", "config/master.key", "config/application.yml", "config/puma/production.rb"

# Depending on your own configuration
append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', '.bundle', 'public/system', 'public/uploads'

set :rbenv_type, :user
set :rbenv_ruby, File.read('.ruby-version').strip

set :rbenv_prefix, "RBENV_ROOT=#{fetch(:rbenv_path)} RBENV_VERSION=#{fetch(:rbenv_ruby)} #{fetch(:rbenv_path)}/bin/rbenv exec"
set :rbenv_map_bins, %w{rake gem bundle ruby rails}
set :rbenv_roles, :all # default value

set :migration_role, :app

# Whenever config
set :whenever_environment, fetch(:stage)
set :whenever_identifier, "#{fetch(:application)}_#{fetch(:stage)}"
set :whenever_variables, -> do
  "'environment=#{fetch :whenever_environment}" \
  "&rbenv_root=#{fetch :rbenv_path}'"
end

namespace :deploy do
  after :publishing, 'puma:restart'
  after :finishing, :cleanup
end

namespace :puma do
  task :restart do
    on roles :web do
      execute 'sudo systemctl restart puma.service'
    end
  end

  task :start do
    on roles :web do
      execute 'sudo systemctl start puma.service'
    end
  end

  task :stop do
    on roles :web do
      execute 'sudo systemctl stop puma.service'
    end
  end
end
```

whenever schedule.rb sample

```ruby
if @rbenv_root
  job_type :rake,    %{cd :path && :environment_variable=:environment :rbenv_root/bin/rbenv exec bundle exec rake :task --silent :output}
  job_type :runner,  %{cd :path && :rbenv_root/bin/rbenv exec bundle exec rails runner -e :environment ':task' :output}
  job_type :script,  %{cd :path && :environment_variable=:environment :rbenv_root/bin/rbenv exec bundle exec script/:task :output}
end

set :output, "#{path}/log/cron.log"

```

logrotate sample

```conf
# sudo /etc/logrotate.d/your-app

/home/ubuntu/app/your-app/shared/log/*.log {
  daily
  missingok
  rotate 14
  compress
  delaycompress
  dateext
  notifempty
  copytruncate
  create 0640 ubuntu ubuntu
  sharedscripts
  postrotate
    kill -HUP `cat /home/deploy/app/your-app/shared/tmp/pids/puma.pid`
    invoke-rc.d nginx rotate >/dev/null 2>&1
  endscript
}
```

切换 npm 和 yarn 源
国内服务器每次部署会卡在 `[4/4] Building fresh packages...` 这一步，调整下面三个源可以解决

```bash
$ npm config set registry https://registry.npm.taobao.org
$ yarn config set registry https://registry.npm.taobao.org
$ yarn config set sass_binary_site http://cdn.npm.taobao.org/dist/node-sass -g 
```

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
