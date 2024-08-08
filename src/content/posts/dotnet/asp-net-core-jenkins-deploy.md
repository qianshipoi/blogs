---
title: Asp.Net Core Jenkins Deploy
published: 2024-04-08
description: ''
image: ''
tags: ['Dotnet','Jenkins']
category: 'Develop'
draft: false
---
## 1.下载安装 jenkins

[Windows安装Jenkins详细教程（图文教程） - 小测试圈 - 博客园 (cnblogs.com)](https://www.cnblogs.com/caoyunpu/p/16711111.html)



## 2.安装配置 Publish over SSH 插件

服务器安装 OpenSSH：[PowerShell/Win32-OpenSSH: Win32 port of OpenSSH (github.com)](https://github.com/PowerShell/Win32-OpenSSH)

安装：Dashboard -> Manage Jenkins -> Plugins -> Available plugins -> 搜索 `Publish over SSH` 安装

配置：Dashboard -> Manage Jenkins -> System -> Publish over SSH -> SSH Servers -> hostname 填写服务器地址 -> 高级 -> 填写 password -> Test Configuration（Success 即为配置成功）

>remote directory 在 windows 服务器下需要在路径前面加上 `/`



## 3.新建任务

Dashboard -> 新建Item -> 输入名称选择 Freestyle project -> 源代码管理选择Git -> 填写完URL 后添加 Credentials -> 指定要监听的分支 -> 构建触发器选择 Poll SCM -> 日程表填入 `H/2 * * * *` 没两分钟检查是否有更新 -> Build Steps -> 添加 `Execute Windows batch command`  -> 填写命令1 -> 添加 `Send files or execute commands over SSH` -> 选择服务器 ->  高级勾选 `Verbose output in console` -> `Transfer Set` 填写表单1  -> 保存 -> Build Now



>命令1
>
>```powershell
>dotnet resoter
>dotnet build
>dotnet publish 工程名称 -o .\publish
>```
>
>表单1
>
>Sources files = \*\*/publish/\*
>
>Remove prefix = /publish
>
>Remote directory = test
>
>Exec command = C:/cicd/auto-publish.bat



> auto-publish.bat

```powershell
cd /d %~dp0
::获取当前目录

@set WebSiteName="网站名称"
@set SCWebApiWebPoolName="应用程序池名称"
@set CicdDir="新程序发布文件夹"
@set TargetDir="程序文件夹"

::停止一下IIS站点
@echo stop WebSite start...
C:\Windows\System32\inetsrv\appcmd.exe stop site %WebSiteName%
@echo stop WebSite finished...

::停止应用程序池
@echo stop WebSite start...
C:\Windows\System32\inetsrv\appcmd.exe stop apppool /apppool.name:%SCWebApiWebPoolName%
@echo stop WebSite finished...

::复制文件
xcopy %CicdDir% %TargetDir% /f/y

::启动应用程序池
@echo stop WebSite start...
C:\Windows\System32\inetsrv\appcmd.exe start apppool /apppool.name:%SCWebApiWebPoolName%
@echo stop WebSite finished...

::启动IIS站点
@echo Restart WebSite start...
C:\Windows\System32\inetsrv\appcmd.exe start site %WebSiteName%
@echo Restart WebSite finished...

```

