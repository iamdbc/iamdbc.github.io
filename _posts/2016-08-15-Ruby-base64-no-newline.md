---
layout: post
title:  "Ruby base64 不要 \"\\n\" 结尾"
date:   2016-08-15 20:20:00 +0800
categories: articles
keywords: ruby base64 no new line
---

今天用到ruby的base64加密，在另一端decode的时候提示错误，后来发现是Base64.encode64(string)的时候自动给末尾加了"\n"换行导致的

查了手册以后发现还有一个方法，Base64.strict_encode64(string)，使用这个就不会有"\n"了
