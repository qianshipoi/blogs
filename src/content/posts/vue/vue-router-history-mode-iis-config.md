---
title: vue/vue-router-history-mode-iis-config
published: 2021-07-25
description: ''
image: ''
tags: ['Vue', 'IIS']
category: 'Develop'
draft: false
---

1. 安装插件：https://www.iis.net/downloads/microsoft/url-rewrite

 2. 配置web.config

>     <?xml version="1.0" encoding="UTF-8"?>
>     <configuration>
>         <system.webServer>
>             <staticContent>
>                 <mimeMap fileExtension=".woff" mimeType="font/x-font-woff" />
>             </staticContent>
>     	      <rewrite>
>                 <rules>
>                   <rule name="API Rule" stopProcessing="true">
>                     <match url="^(api|account|manage)(.*)$" />
>                     <action type="None" />
>                   </rule>
>                   <rule name="Angular Rule" stopProcessing="true">
>                     <match url="(.*)" />
>                     <conditions logicalGrouping="MatchAll">
>                       <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
>                       <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
>                     </conditions>
>                     <action type="Rewrite" url="/" />
>                   </rule>
>                 </rules>
>             </rewrite>
>         </system.webServer>
>     </configuration>
>
>     **注意事项：嵌套路由要将 vue.config.js 中 publicPath 修改为 './'**
