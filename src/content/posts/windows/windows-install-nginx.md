---
title: Windows Install Nginx
published: 2022-06-13
description: ''
image: ''
tags: ['Windows', 'Nginx']
category: 'Develop'
draft: false
---

## 1.下载文件 [文件地址](http://nginx.org/en/download.html)

## 2.解压后双击nginx.exe，浏览器输入localhost:80后出现nginx欢迎页面说明成功启动。

## 3.任务管理器关闭nginx.exe，将nginx注册为windows服务。下载系统服务[Server](https://github.com/winsw/winsw/releases)。

## 4.将服务程序从命名为nginx-server.exe 。新建nginx-server.xml，内容为：

```xml
<service>
<id>nginx</id>
<name>nginx</name>
<description>nginx-server</description>
<logpath>C:webappsNginxnginx-1.18.0logs</logpath>
<logmode>roll</logmode>
<executable>C:webappsNginxnginx-1.18.0nginx.exe</executable>
</service>
```

## 5.CMD执行nginx-server install将其注册为系统服务。
