---
title: Mosquitto 服务无法启动：密码文件权限问题解决方案
published: 2026-01-15
description: '使用 mosquitto_passwd 生成密码文件后，Windows 服务无法启动？本文教你如何通过配置 SYSTEM 用户权限解决这个问题。'
image: ''
tags: [Mosquitto, MQTT, Windows, 权限问题]
category: 'Dev'
draft: false
---

## 问题描述

在 Windows 系统上安装并配置 Mosquitto MQTT 服务器时，使用 `mosquitto_passwd.exe` 生成密码文件并在配置文件中指定密码文件位置后，服务却无法正常启动。

### 环境信息

- 操作系统：Windows
- 软件：Mosquitto MQTT Broker
- 问题：服务启动失败

### 配置步骤

1. 使用 `mosquitto_passwd.exe` 创建密码文件：

```bash
mosquitto_passwd.exe -c passwordfile.txt username
```

2. 在 `mosquitto.conf` 配置文件中添加密码文件路径：

```conf
password_file C:\path\to\passwordfile.txt
allow_anonymous false
```

3. 尝试启动 Mosquitto 服务，但服务启动失败。

## 问题分析

Mosquitto 在 Windows 上作为服务运行时，默认使用 **SYSTEM** 账户。当我们手动创建密码文件时，该文件的权限通常只授予给当前用户，而 SYSTEM 账户没有读取该文件的权限。

因此，当 Mosquitto 服务尝试读取密码文件时，由于权限不足导致服务启动失败。

## 解决方案

解决这个问题的关键是为 SYSTEM 账户添加密码文件的读取权限。

### 步骤 1：找到密码文件

找到你创建的密码文件位置，例如 `C:\mosquitto\passwordfile.txt`。

### 步骤 2：添加 SYSTEM 权限

1. 右键点击密码文件，选择 **属性**
2. 切换到 **安全** 选项卡
3. 点击 **编辑** 按钮
4. 点击 **添加** 按钮
5. 在"输入对象名称来选择"框中输入：`SYSTEM`
6. 点击 **检查名称**，确认对象名称正确
7. 点击 **确定**
8. 在权限列表中，为 SYSTEM 用户勾选 **读取** 和 **读取和执行** 权限
9. 点击 **应用** 和 **确定**

### 步骤 3：重启服务

通过以下方式重启 Mosquitto 服务：

**使用服务管理器：**
1. 按 `Win + R`，输入 `services.msc`
2. 找到 Mosquitto Broker 服务
3. 右键选择 **重新启动**

**使用命令行：**

```powershell
# 停止服务
net stop mosquitto

# 启动服务
net start mosquitto
```

或使用 PowerShell：

```powershell
Restart-Service -Name "Mosquitto Broker"
```

## 验证

服务启动后，可以通过以下方式验证：

1. 检查服务状态：

```powershell
Get-Service -Name "Mosquitto Broker"
```

2. 使用 MQTT 客户端测试连接：

```bash
mosquitto_sub -h localhost -t test -u username -P password
```

## 总结

在 Windows 上配置 Mosquitto 密码认证时，需要注意文件权限问题。由于服务以 SYSTEM 账户运行，必须确保 SYSTEM 有权读取密码文件。通过为 SYSTEM 账户添加适当的文件权限，可以快速解决服务无法启动的问题。

### 预防措施

为避免类似问题，建议：

1. 将 Mosquitto 配置文件和密码文件放在 Mosquitto 安装目录下
2. 在创建配置文件后，立即检查并配置 SYSTEM 权限
3. 使用具有管理员权限的账户创建和配置文件
4. 定期检查服务日志，及时发现权限相关错误

### 相关资源

- [Mosquitto 官方文档](https://mosquitto.org/documentation/)
- [mosquitto_passwd 使用说明](https://mosquitto.org/man/mosquitto_passwd-1.html)
- [Windows 服务账户权限管理](https://learn.microsoft.com/zh-cn/windows/security/identity-protection/access-control/)
