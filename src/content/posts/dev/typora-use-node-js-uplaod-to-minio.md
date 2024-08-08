---
title: Typora Use NodeJS Upload To Minio
published: 2021-07-23
description: ''
image: ''
tags: ['Typora', 'NodeJS', 'Minio']
category: 'Develop'
draft: false
---

Typora 使用 Nodejs 上传到 Minio 服务器

1. Typora  文件 -> 偏好设置 -> 图像 -> 上传服务（Custom Command）命令（node [path] name）

2. 电脑上新建Upload.js 文件（npm install Minio fs）

   ```js
   /*
    * typora插入图片调用此脚本，上传图片到图床
    */

   const path = require('path')
   // minio for node.js
   const Minio = require('minio')
   const { promises } = require('fs')

   // 解析参数， 获取图片的路径，有可能是多张图片
   const parseArgv = () => {
     const imageList = process.argv.slice(3).map(u => path.resolve(u))
     return imageList
   }

   // 入口
   const uploadImageFile = async (imageList = []) => {
     // 创建连接
     const minioClient = new Minio.Client({
       // 这里填写你的minio后台域名
       endPoint: 'minio.kuriyama.top',
       port: 443,
       useSSL: true,
       // 下面填写你的accessKey和secretKey
       accessKey: 'xxxx',
       secretKey: 'xxxx'
     })

     // 开始上传图片
     const metaData = {}
     const tasks = imageList.map(image => {
       return new Promise(async (resolve, reject) => {
         try {
           // 图片重命名，这里采用最简单的，可以根据自己需求重新实现
           const name = `${process.argv[2]}/${new Date().Format('yyyyMMHH')}/${Date.now()}${path.extname(image)}`
           // 具体请看Minio的API文档，这里是将图片上传到blog这个bucket上
           const res = await minioClient.fPutObject('typora', name, image, metaData)
           resolve(name)
         } catch (err) {
           reject(err)
         }
       })
     })

     const result = await Promise.all(tasks)

     // 返回图片的访问链接
     result.forEach(name => {
       const url = `https://minio.kuriyama.top/typora/${name}`
       // Typora会提取脚本的输出作为地址，将markdown上图片链接替换掉
       console.log(url)
     })
   }

   Date.prototype.Format = function (fmt) {
     var o = {
       'M+': this.getMonth() + 1,
       'd+': this.getDate(),
       'H+': this.getHours(),
       'm+': this.getMinutes(),
       's+': this.getSeconds(),
       'S+': this.getMilliseconds()
     }
     //因为date.getFullYear()出来的结果是number类型的,所以为了让结果变成字符串型，下面有两种方法：
     if (/(y+)/.test(fmt)) {
       //第一种：利用字符串连接符“+”给date.getFullYear()+''，加一个空字符串便可以将number类型转换成字符串。
       fmt = fmt.replace(RegExp.$1, (this.getFullYear() + '').substr(4 - RegExp.$1.length))
     }
     for (var k in o) {
       if (new RegExp('(' + k + ')').test(fmt)) {
         //第二种：使用String()类型进行强制数据类型转换String(date.getFullYear())，这种更容易理解。
         fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (('00' + o[k]).substr(String(o[k]).length)))
       }
     }
     return fmt
   }

   // 执行脚本
   uploadImageFile(parseArgv())
   ```

3. Minio 服务器使用 `Nginx` 代理需要配置

   ```nginx
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header X-Forwarded-Proto $scheme;
   proxy_set_header Host $http_host;
   proxy_http_version 1.1;
   ```



