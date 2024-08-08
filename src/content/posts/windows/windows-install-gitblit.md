---
title: Windows Install Gitblit
published: 2021-07-23
description: ''
image: ''
tags: ['Windows', 'Gitblit']
category: 'Develop'
draft: false
---

Gitblit安装步骤（需先安装 `Java` 环境）：

1. Gitblit[官网](http://gitblit.github.io/gitblit/) 下载gitblit文件。
2. 文件解压后，使用 `VS Code` 打开。
3. 将 `data/defaults.properties` 文件复制到更目录，并重命名为 `my.properties`。
4. 打开 `data/gitblit.properties` 文件，注释 `include = defaults.properties`,添加 `include = my.properties`。
5. 在 `my.properties` 中找到 `server.httpPort` 修改程序访问端口。
6. `my.properties` 中 `server.httpsBindInterface` 可以绑定IP地址。
7. 控制台使用 `.\gitblit.cmd` 启动程序。
8. 使用浏览器访问该网站，默认用户名和密码是 `admin`。
9. 将 `gitblit` 设置为 Windows 服务：
   - 下载 [`prunsrv.exe`](http://archive.apache.org/dist/commons/daemon/binaries/windows/) 程序。
   - 打开 `installService.cmd` 文件，添加 `SET CD=当前文件夹所在目录`，将 `"%CD%\%ARCH%\gitblit.exe"` 改为 `"%CD%\%ARCH%\prunsrv.exe"`。
   - 打开 `uninstallService.cmd` 文件，同上。

