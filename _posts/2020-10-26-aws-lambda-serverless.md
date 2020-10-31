---
layout: post
title:  "serverless framework 简易教程"
date:   2020-10-26 20:50:00 +0800
categories: serverless
keywords: serverless lambda aws
comments: true
---

这是一篇简易教程，按照这个教程可以快速的把项目跑起来

如果你看到这篇文章，说明你也遇到 lambda 开发中的一些问题，例如本地开发，测试，部署，打包，第三方扩展等问题

为了统一开发环境，所以需要引入 template，让每个 developer 使用相同的 template，保证环境和配置都相同

AWS 提供了一套 serverless 框架 SAM(https://aws.amazon.com/cn/serverless/sam/)，但是配置上比较复杂

这里是使用了另一个框架 serverless framework，下面主要介绍一下 serverless.yml 的配置，本地测试，以及部署的方式

## 安装

### 安装 serverless framework
```bash
$ npm install -g serverless
```

### 安装 layer plugin
为了在 serverless framework 中使用 layer，需要在代码所在目录中安装 plugin
```bash
$ npm install -D serverless-layers
```

### 配置 aws credentials
本地测试和部署时，会用到 aws 权限，需要在本地配置 aws cli 和 credential
https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html

本地可以配置多个 aws 的 profile，文档里有说明

## 使用

### 配置 serverless.yml

把 `serverless.yml` 文件放入你的 lambda 目录中

```yml
service: 多个 function 可以归于一个 service，便于管理，必填

provider:
  name: aws (必填，serverless framework 可用于多个云厂商，根据不同厂商会自动进行配置)
  runtime: nodejs12.x (Lambda runtime)
  stage: ${opt:stage, 'dev'} (运行环境，默认 'dev')
  memorySize: 1024 (选填, Lambda 执行时的内存)
  timeout: 60 (选填, 超时时间)
  region: ap-northeast-1 (选填，Lambda 部署地区)
  role: arn:aws:iam::xxxxx (Lambda 执行时的 role)
  deploymentBucket:
    name: bucketName (Lambda 代码和 layer 保存在 s3 的 bucket，多个 lambda 可以放在相同的 bucket，会根据 service 自动创建目录)
  environment: （环境变量)
    REDIS_CONN_URL: ${self:custom.redis_conn_url.${self:provider.stage}} (根据不同的 stage 使用/设置不同的变量)
    REDIS_PASSWORD: ${self:custom.redis_password.${self:provider.stage}}

package: (部署代码时要 exclude 或 include 的代码，本地有一些测试文件例如 event.json 等不需要部署)
  exclude:
    - ./**
  include:
    - index.js

functions:
  yourFunctionName: (函数名)
    handler: index.handler (函数执行入口)
    vpc: (vpc 配置)
      securityGroupIds:
        - sg-xxx
      subnetIds:
        - subnet-xxx1
        - subnet-xxx2
    layers:
      - arn:aws:lambda:ap-northeast-1:layer1
      - arn:aws:lambda:ap-northeast-1:layer2
    events: (event 事件触发 lamabda)
      - s3:
          bucket: ${self:custom.event_bucket.${self:provider.stage}} (触发的 bucket)
          event: s3:ObjectCreated:* (触发的事件)
          rules: (规则限制，例如触发的 object 的 prefix 和 suffix)
            - prefix: ${self:custom.event_bucket_prefix.${self:provider.stage}}
            - suffix: .json
          existing: true (bucket 存在时设置为 true，否则会自动创建 bucket)

plugins: (要使用 layer 时要加上该 plugin)
  - serverless-layers
  - serverless-ruby-layer (runtime 为 ruby 时使用, nodejs 不需要)

custom: (自定义变量)
  event_bucket:
    dev: dev
    production: production
  event_bucket_prefix:
    dev: dev
    production: production
  redis_conn_url:
    dev: dev
    production: production
  redis_password:
    dev: dev
    production: production

```

### 使用

以 serverless framework s3 event 为例

```bash
$ serverless invoke local --function yourFunctionName -p event.json --aws-profile your-aws-profile
```

- -p 后面是输入到 function 的 json 文件的路径
- event.json 是触发 s3 event 的 json 数据，event 的格式可以在 Lambda 中根据不同的需要创建，之后再修改为要测试的具体内容
- --aws-profile 是本地设置的 aws 授权账号的名称，为了避免误操作，不加 --aws-profile 的情况下使用默认账号

在本地代码目录下执行上面的命令，只会在本地执行代码，不会进行任何部署，但是会根据代码逻辑操作 aws 资源，例如读取 s3 object 或写入等

## 部署
```bash
# 默认部署会在 functionName 中加入 -dev，同时使用 dev 的环境变量
$ serverless deploy --aws-profile your-aws-profile

# 指定 stage 为 production 进行部署，会在 functionName 中加入 -production，同时使用 production 的环境变量
$ serverless deploy --stage production --aws-profile your-aws-profile
```

参考
https://www.serverless.com/framework/docs/providers/aws/guide/serverless.yml/


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
