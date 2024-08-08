---
title: Asp.Net Core Download big file error issue
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

## 问题描述：

> 使用HttpClient下载文件抛出异常：Cannot write more bytes to the buffer than the configured maximum buffer size: 2147483647.
## 解决方式：
> 在GetAsync方法第二个参数加上 HttpCompletionOption.ResponseHeadersRead （获取到响应头就返回）
> ``` var response = await client.GetAsync(url, HttpCompletionOption.ResponseHeadersRead) ```
