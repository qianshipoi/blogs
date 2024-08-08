---
title: Docker Install Nginx
published: 2022-12-05
description: ''
image: ''
tags: ['Docker', 'Nginx']
category: 'Develop'
draft: false
---

```shell
docker pull nginx:latest
```

```shell
mkdir conf.d
mkdir html
mkdir log
touch nginx.conf
vim nginx.conf
```

粘贴

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```



```shell
docker run -d --name nginx --net=host --restart=always -v /root/apps/nginx_docker/nginx.conf:/etc/nginx/nginx.conf -v /root/apps/nginx_docker/conf.d:/etc/nginx/conf.d -v /root/apps/nginx_docker/html:/usr/share/nginx/html -v /root/apps/nginx_docker/log:/var/log/nginx --privileged=true nginx
```



-d
以守护式容器在后台运行
–name nginx
指定容器名称
–net=host
1、添加以后就不需要再做端口映射了.比如docker容器内在8080端口起了一个web server.不加的话需要把本机的某个port比如7979和docker内的8080做一个映射关系,访问的时候访问7979. 加了net=host则直接访问8080.；
2、另外,加了net=host后会使得创建的容器进入命令行好名称显示为主机的名称而不是一串id；
3、能够直接使用127.0.0.1或localhost连接本机器
–restart=always
使我们在重启docker时，自动启动相关容器。
–privileged=true
设置特权级运行的容器
