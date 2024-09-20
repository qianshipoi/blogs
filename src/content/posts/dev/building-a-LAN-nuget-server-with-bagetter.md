---
title: Building a LAN nuget server with bagetter
published: 2024-08-08
description: ''
image: ''
tags: ['Bagetter', 'Nuget', 'IIS']
category: 'Develop'
draft: false
---

使用 `bagetter` 搭建局域网 `nuget` 服务器

开源地址：[bagetter/BaGetter: A lightweight NuGet and symbol server (github.com)](https://github.com/bagetter/BaGetter)

下载地址：[Releases · bagetter/BaGetter (github.com)](https://github.com/bagetter/BaGetter/releases)

文档地址：[Index | BaGetter](https://www.bagetter.com/docs)

Dotnet SDK：[下载 .NET(Linux、macOS 和 Windows) (microsoft.com)](https://dotnet.microsoft.com/zh-cn/download)

1. 下载 `bagetter` 并解压缩。下载 Dotnet SDK 最新版安装。
2. 打开 `IIS` 添加网站：物理路径选择解压缩目录，IP地址分配本机IP，端口随意。
3. 打开添加的网站，在处理程序映射内找到 `WebDAV` 相关模块删除或禁用，在模块内找到 `webDAV` 相关模块删除或禁用（防止`IIS` 拦截 `PUT` 请求）。
4. 打开应用程序池找到 `bagetter` 设置为无托管代码。如果有权限问题将高级设置的 `Identity` 设置为 `LocalSystem`。
5. 打开 `windows 防火墙` ，在高级设置内添加入站规则：选择端口 -> TCP + 特定本地端口：网站端口 -> 后续默认即可。

