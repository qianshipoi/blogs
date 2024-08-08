---
title: Asp.Net Core CORS issue
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

# 1 安装 Microsoft.AspNet.WebApi.Cors 程序包

> **右键项目-->管理 NuGet 程序包 --> 浏览输入 CORS --> 安装 Microsoft.AspNet.WebApi.Cors。**

# 2 添加配置项

> **在 App_Start 文件夹内 WebApiConfig.cs 文件中 Register 方法中 添加  config.EnableCors(new EnableCorsAttribute("\*", "\*", "\*"));**

# 3 添加跨域特性

> ** 在需要进行跨域操作的 ApiController 前加  [EnableCors("\*", "\*", "\*")] **
