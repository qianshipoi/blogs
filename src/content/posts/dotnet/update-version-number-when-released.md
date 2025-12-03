---
title: dotnet/update-version-number-when-released
published: 2025-12-03
description: '构建时自动更新版本号'
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

在 `PropertyGroup` 节点中添加
```csharp
<PropertyGroup>
	<Version>
		1.0.$([System.Math]::Floor($([System.DateTime]::UtcNow.Subtract($([System.DateTime]::Parse('2000-01-01T00:00:00Z'))).TotalDays))).$([MSBuild]::Divide($([System.Math]::Floor($([System.DateTime]::UtcNow.TimeOfDay.TotalSeconds))), 2))
	</Version>
</PropertyGroup>
```

最终生成的版本号示例： 1.0.9238.28518

其中，Major 与 Minor 是固定的，Build 是2000年1月1日至今的天数，Revision 是今天的秒数 / 2 所得的值。（为了防止数值超过 65535）

获取版本号
```csharp
internal static class VersionHelper
{
    internal static string GetInformationalVersion()
    {
        var asm = Assembly.GetEntryAssembly() ?? Assembly.GetExecutingAssembly();
        return asm.GetCustomAttribute<AssemblyInformationalVersionAttribute>()?.InformationalVersion ?? "unknown";
    }

    internal static string GetFileVersion()
    {
        var asm = Assembly.GetEntryAssembly() ?? Assembly.GetExecutingAssembly();
        return asm.GetCustomAttribute<AssemblyFileVersionAttribute>()?.Version ?? "unknown";
    }

    internal static string GetVersion()
    {
        var asm = Assembly.GetEntryAssembly() ?? Assembly.GetExecutingAssembly();
        return asm.GetName().Version?.ToString() ?? "unknown";
    }

    internal static DateTime GetBuildDateUtc()
    {
        var asm = Assembly.GetEntryAssembly() ?? Assembly.GetExecutingAssembly();
        var version = asm.GetName().Version;
        return new DateTime(2000, 1, 1).AddDays(version.Build).AddSeconds(version.Revision * 2);
    }
}
```

代码来源：https://xoyozo.net/Blog/Details/net-core-auto-updates-version-with-each-release
