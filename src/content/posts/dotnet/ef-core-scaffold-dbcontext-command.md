---
title: EF Core scaffold-dbcontext Command
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

工具的scaffold-dbcontext（数据库上下文脚手架）指令来生成models和context。

指令详细介绍：

Scaffold-DbContext [-Connection] <String> [-Provider] <String> [-OutputDir <String>] [-Context <String>]
[-Schemas <String>] [-Tables <String>] [-DataAnnotations] [ -Force] [-Project <String>]
[-StartupProject <String>] [-Environment <String>] [<CommonParameters>]

PARAMETERS
-Connection <String>
指定数据库的连接字符串。

-Provider <String>
指定要使用的提供程序。例如，Microsoft.EntityFrameworkCore.SqlServer。

-OutputDir <String>
指定用于输出类的目录。如果省略，则使用顶级项目目录。

-Context <String>
指定生成的DbContext类的名称。

-Schemas <String>
指定要为其生成类的模式。

-Tables <String>
指定要为其生成类的表。

-DataAnnotations [<SwitchParameter>]
使用DataAnnotation属性在可能的情况下配置模型。如果省略，输出代码将仅使用流畅的API。

-Force [<SwitchParameter>]
强制脚手架覆盖现有文件。否则，只有在没有输出文件被覆盖的情况下，代码才会继续。

-Project <String>
指定要使用的项目。如果省略，则使用默认项目。

-StartupProject <String>
指定要使用的启动项目。如果省略，则使用解决方案的启动项目。

-Environment <String>
指定要使用的环境。如果省略，则使用“开发”。

原文：[https://blog.csdn.net/liangyely/article/details/87522867][1]


[1]: https://blog.csdn.net/liangyely/article/details/87522867
