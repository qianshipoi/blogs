---
title: VS Designer Cache To Big Issue
published: 2023-04-13
description: ''
image: ''
tags: ['VirtualStudio', 'Windows']
category: 'Develop'
draft: false
---

# VS Designer Cache 文件夹会缓存项目输出目录的文件，导致C盘被占用。

解决方案：设置软链接

```shell
mklink /d C:\Users\xxx\AppData\Local\Microsoft\VisualStudio\17.0_a0d456fc\Designer\Cache D:\dev\vs-designer-cache
```

`/d` 文件夹映射

第一个路径：原缓存文件夹（原路径在C盘）

第二个路径：要映射的路径

