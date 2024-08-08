---
title: Windwos Install Minio
published: 2022-06-13
description: ''
image: ''
tags: ['Windows', 'Minio']
category: 'Develop'
draft: false
---

## 1. 下载Minio文件 [Minio官网](https://dl.min.io/server/minio/release/windows-amd64/minio.exe)      [Windows-Server](https://dl.min.io/server/minio/release/windows-amd64/minio.exe)

## 2.下载windows服务注册文件 [Winsw](https://github.com/winsw/winsw/releases) 并重命名为minio-server.exe

## 3.同目录下创建minio-server.xml文件，与服务文件名相同

## 4.配置xml文件 配置规则

    <service>
        <id>minio-server</id>
        <name>minio-server</name>
        <description>minio文件存储服务器</description>
        <!-- 可设置环境变量 -->
        <env name="HOME" value="%BASE%"/>
        <executable>%BASE%minio.exe</executable>
        <arguments>server "%BASE%data"</arguments>
        <!-- <logmode>rotate</logmode> -->
        <logpath>%BASE%logs</logpath>
        <log mode="roll-by-size-time">
        <sizeThreshold>10240</sizeThreshold>
        <pattern>yyyyMMdd</pattern>
        <autoRollAtTime>00:00:00</autoRollAtTime>
        <zipOlderThanNumDays>5</zipOlderThanNumDays>
        <zipDateFormat>yyyyMMdd</zipDateFormat>
     </log>
    </service>

## 5. cmd执行minio-server.exe install 安装服务

## 6. 安装完成后启动服务，成功就能正常使用了，web管理地址为127.0.0.1:9000

## 7.修改密码 到项目目录下 .minio.sys/config/config.json 文件内修改密码 搜索 【access_key】和【secret_key】 value 为 账号和密码
