---
title: Asp.Net Core IIS Upload big file error issue
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

> iis配置
>      // 最大为2147483648 2GB
>      <security>
>       <requestFiltering>
>         <requestLimits maxAllowedContentLength="2072576000"/>
>       </requestFiltering>
>     </security>

> Startup
> // 设置最大长度
> services.Configure<FormOptions>(x => {
>            x.ValueLengthLimit = int.MaxValue;
>            x.MultipartBodyLengthLimit = long.MaxValue;
>            x.MemoryBufferThreshold = int.MaxValue;
>        });
> 控制器

      RequestSizeLimit(long.MaxValue)
