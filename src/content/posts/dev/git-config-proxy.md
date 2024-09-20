---
title: Git 配置代理
published: 2024-09-20
description: ''
image: ''
tags: ['Git']
category: 'Develop'
draft: false
---

### 全局代理

``` shell
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

### 针对Github
``` shell
# 使用socks
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080

# 使用http代理
git config --global http.https://github.com.proxy 'http://127.0.0.1:7890'

# 取消代理
git config --global --unset http.https://github.com.proxy

```
