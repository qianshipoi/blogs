---
title: dotnet/xunit-test-pack-path-exception
published: 2025-02-19
description: ''
image: ''
tags: ['Dotnet', 'WPF']
category: 'Develop'
draft: false
---

使用`xUnit`对`WPF`类库进行测试时，发现使用`FontFamily`加载字体资源时提示`System.UriFormatException : Invalid URI: Invalid port specified.`。

是由于`pack UriScheme`是在 `System.Windows.Application` 构建时注册的，解决方案：在启动测试前检查当前是否支持 `pack://`，如果不支持则创建 `System.Windows.Application`。

```C#
public AMapWeatherImageProviderTest()
{
    if (!UriParser.IsKnownScheme("pack"))
        new System.Windows.Application();
}
```
