---
title: Windows Install OpenSSH
published: 2023-06-07
description: ''
image: ''
tags: ['Windows', 'OpenSSH']
category: 'Develop'
draft: false
---

## 安装 OpenSSH

> [Releases · PowerShell/Win32-OpenSSH (github.com)](https://github.com/PowerShell/Win32-OpenSSH/releases)  下载 OpenSSH


## 将 PowerShell 设置为 SSH 默认 Shell

> [适用于 Windows 的 OpenSSH 服务器配置 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_server_configuration#configuring-the-default-shell-for-openssh-in-windows)

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -PropertyType String -Force
```
