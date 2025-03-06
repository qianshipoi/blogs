---
title: Build nodejs program using pkg
published: 2025-03-06
description: ''
image: ''
tags: ['NodeJs']
category: ''
draft: false
---

> pkg: https://github.com/yao-pkg/pkg <br/>
> pkg-fetch: https://github.com/yao-pkg/pkg-fetch

> 安装pkg: `npm install -g @yao-pkg/pkg`;

#### 在 `package.json` 文件中添加
```json
{
  "name": "glb-optimize",
  "version": "1.0.0",
  "type": "commonjs",
  "main": "src/index-common.js",
  "pkg": {
    "scripts": "src/index-common.js", // 入口文件
    "assets": [
      "node_modules/@gltf-transform/core/**/*",
      "node_modules/@gltf-transform/extensions/**/*",
      "node_modules/@gltf-transform/functions/**/*",
      "node_modules/sharp/**/*",
      "node_modules/@img/**/*" // 需包含的三方包
    ],
    "targets": [
      "node22-win-x64" // 构建目标版本
    ],
    "outputPath": "dist", // 输出路径
    "public": true
  },
  "scripts":{
    "build": "pkg ."
  }
}
```
> 执行 `npm run build` 后会 `dist` 目录生成 `package.json` 内指定的 `{name}.exe`。
>
> 构建过程中会去 `pkg-fetch` 获取匹配的 `node` 包，很大概率会下载失败，下载失败时可以去 [pack-fetch/releases](https://github.com/yao-pkg/pkg-fetch/releases) 找到对应的文件，例如 `node22` 下载 `node-v22.13.1-win-x64`，并将下载好的文件放入 `C:\\Users\\{用户名}\\.pkg-cache\\{version}\` 目录下，重命名为 `fetched-v22.13.1-win-x64`。
