---
layout: post
comments: true
title:  "ruby 报错 progress in another thread"
date:   2020-01-25 19:10:00 +0800
categories: ruby
---

有一天启动rails时遇到了下面的报错，重新安装ruby也不起作用

最后是升级了ruby version解决的，报错信息如下

```bash
+[__NSPlaceholderDictionary initialize] may have been in 
progress in another thread when fork() was called
+[__NSPlaceholderDictionary initialize] may have been in 
progress in another thread when fork() was called. We cannot safely 
call it or ignore it in the fork() child process. Crashing instead. Set 
a breakpoint on objc_initializeAfterForkError to debug
```
