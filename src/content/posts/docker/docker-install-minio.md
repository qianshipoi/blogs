---
title: Docker Install Minio
published: 2022-12-05
description: ''
image: ''
tags: ['Docker','Minio']
category: 'Develop'
draft: false
---

## Docker run command：

>`MINIO_ROOT_USER` ：用户名。
>
>`MINIO_ROOT_PASSWORD` ：密码。
>
>`MINIO_BROWSER_REDIRECT_URL`：浏览器重定向地址 控制台外部访问地址（如果不设置，分享的文件地址为docker内网地址）。
>
>`MINIO_SERVER_URL` ：服务器外部访问地址。
>
>`console-address` ：控制台地址。
>
>`address` ：服务器地址。

```shell
docker run -d
	--name=minio
	--env=MINIO_ROOT_USER=xxxx
	--env=MINIO_ROOT_PASSWORD=xxxx
	--env=MINIO_BROWSER_REDIRECT_URL=https://minio-console.kuriyama.top
	--env=MINIO_SERVER_URL=https://oss.kuriyama.top
	--volume=/root/minio/data/:/data/
	--volume=/root/minio/config/:/root/.minio
	--volume=/data -p 9000:9000 -p 9001:9001
	--restart=always
	minio/minio server /data --console-address ":9001"
	--address ":9000"
```



## ssl config：

>在 `/root/minio/config/certs` 目录下放置 ` https://img.kuriyama.top` 相同的证书，并重命名为 `private.key` 和 `public.pem` 即可。

## nginx setting：

>oss.kuriyama.conf

````nginx
server {
    listen  443 ssl;
    server_name  oss.kuriyama.top;
        ssl_certificate ./conf.d/certs/oss.kuriyama.top.pem;
        ssl_certificate_key ./conf.d/certs/oss.kuriyama.top.key;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
    location / {
      proxy_pass http://192.168.0.1:9000;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
server{
        listen 80;
        server_name oss.kuriyama.top;
        return 301 https://$host$request_uri;
}
````

>minio-console.kuriyama.conf

```nginx
server {
   listen  443 ssl;
    server_name  minio-console.kuriyama.top;
        ssl_certificate ./conf.d/certs/minio-console.kuriyama.top.pem;
        ssl_certificate_key ./conf.d/certs/minio-console.kuriyama.top.key;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
           proxy_set_header Host $http_host;
          proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
    location / {
      proxy_pass http://192.168.0.1:9001;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
server{
        listen 80;
        server_name minio-console.kuriyama.top;
        return 301 https://$host$request_uri;
}
```

> 上传大文件提示文件太大，413：nginx设置  `client_max_body_size 100M`
