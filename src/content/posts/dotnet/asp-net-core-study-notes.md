---
title: Asp.Net Core Study Notes
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---


# 1 Progranm类

     1.1: Main 方法 程序入口
     1.2: CreateDefaultBuilder 方法 设置Web服务器 加载主机和应用程序的配置表信息 配置日志记录

# 2 Sartup类

    2.1 ConfigureServices 方法  用于配置应用程序所需要的服务或内容（第三方插件或组件）
    2.2 Configure 方法 用于接收HTTP请求的管道

# 3 .csproj文件 程序集配置文件

    3.1 AspNetCoreHostingModel 标签
    3.1.1 InProcess 在进程内托管
      将应用托管在IIS内直接与HTTP请求进行交互
      CreateDefaultBuilder() 方法调用UserIIS()方法并在IIS工作进程（w3wp.exe或iisexpress.exe）内托管应用程序
      从性能的角度来看，InProcess托管比OutOfProcess托管提供了更高的请求吞吐量。
      System.Diagnostics.Process.GetCurrentProcess().ProcessName; 获取当前进程名称
    3.1.2 OutOfProcess 在进程外托管（默认托管方式）
      将应用托管在Kestrel内直接与HTTP请求进行交互 或者与 IIS、Nginx或Apache 通过代理的方式进行HTTP交互（internet—IIS—Kestrel）
      有两个服务器 内部服务器是 Kestrel 外部服务器是 IIS、Nginx或Apache
      kestrel 是ASP.NEY Core跨平台的Web服务器

# 4 launchSettings.json文件	服务器配置文件

    4.1 iisSettings节点
      4.1.1 windowsAuthentication windows身份验证
      4.1.2 anonymousAuthentication 匿名验证
      4.1.3 iisExpress iis配置项
        4.1.3.1 applicationUrl 配置地址
        4.3.1.2 sslPort 启用ssl 未启用为0
    4.2 profiles节点
      4.2.1 commandName 环境名称
      4.2.2 launchBrowser 是否自动调用浏览器
      4.2.3 environmentVariables 环境变量
       4.2.3.1 ASPNETCORE_ENVIRONMENT 开发环境（Development） 或 生产环境

# 5 appsettings.json文件 程序配置源

    5.1 appsettings.{Environment}.json 不同环境下对应不同托管环境
      5.1.1 User secrets 用户机密	会将信息存在本地
      5.1.2 Environment variables 环境变量(launchSettings.json文件内)
      5.1.3 Command-line arguments 命令行参数
      5.1.3.1 优先级 appsettings.json < appsettings.{Environment}.json < secrets < variables < arguments
    5.2 IConfiguration 获取配置信息的接口


# 6 Middleware 中间件
    6.1 默认自带 Logging StaticFile MVC 三个中间件 处理流程： 用户请求 > 写入请求日志 > 处理静态文件 > MVC进行逻辑处理 > 返回静态文件 > 写入返回日志 > 返回给用户
    6.2 可同时被访问和请求。 可处理请求后，然后将请求传递给下一个中间件。 可以处理请求后，并是管道短路。 可以处理传出响应。 中间件是按照添加顺序执行的。
    6.3 设置编码格式 content.Response.ContentTye = "text/plain;charset=utf-8";
    6.4 app.Use 设置中间件 该方法不会造成短路 可以通过参数next委托调用下一个中间件
    6.5 app.Run 设置中间件 并造成短路 该方法之后的中间件将不会执行
    6.6 中间件执行顺序 第一个中间件走到next 进入下一个中间件 下一个中间件走到next 继续进入下一个中间件 知道中间件发生短路 依次返回给上一个中间键。
    6.7 Middleware1 传入请求 -> Middleware2 传入请求 -> Middleware3 处理请求并产生响应 -> Middleware2 传出响应 -> Middleware1 传出响应

# 7 请求静态文件 ASP.NET Core 默认不支持请求静态文件

    7.1 app.UseDefaultFiles() 定义默认文件 必须在使用静态文件之前注册 默认为 Index.html Index.htm default.html defaule.htm
    7.2 自定义默认文件
      7.2.1 DefaultFilesOptions defaultFilesOption = new DefaultFilesOptions(); 创建对象
      7.2.2 defaultFilesOptions.DefaultFileNmaes.Clear(); 清除默认文件名
      7.2.3 defaultFilesOptions.DefaultFileNames.add("kuriyama.html");  设置默认请求文件名
      7.2.4 app.UseDefaultFiles(defaiultFilesOptions);  将默认文件传入注册事件
    7.3 app.UseStaticFiles();  使用静态文件  之后就能访问项目中wwwroot文件夹内的文件了
    7.4 app.UseDirectoryBrowser(); 注册之后能在浏览器显示项目结构目录  非必要不建议使用
    7.5 ASP.NET Core 请求静态文件综合 app.UseFileServer();   使用此方法就不需要注册那么多中间件
    7.6 自定义默认文件
      7.6.1 FileServerOptions fileServerOptions = new FileServerOptions();
      7.6.2 fileServerOptions.DefaultFilesOptions.DefaultFileNames.Clear();
      7.6.3 fileServerOptions.DefaultFilesOptions.DefaultFileNames.Add("kuriyama.html");
      7.6.4 app.UseFileServer(fileServerOptions);  将默认文件传入注册事件

# 8 开发者异常页面 app.UseDeveloperExceptionPage(); 中间件

    8.1 DeveloperExceptionPageOptions 对象下的 SourceCodeLineCount 属性 设置错误页面显示代码条数

# 9 环境变量 ASPNETCORE_ENVIRONMENT

    9.1 在launchsettings.json 文件中设置环境变量 一般设置开发者环境 Development
    9.2 Staging 和 Production 一般设置在操作系统中
    9.3 在Configure方法中 IHostingEnvironment 接口 中包含环境变量的一些成员
    9.4 如果launchsettings.json 文件中没有配置环境变量 则会在操作系统中寻找 如果系统中也没有则为Production
    9.5 还可以自定义环境变量 使用 env.IsEnvironment()方法判断
