---
title: 使用openssl实现局域网自签证书
published: 2025-04-16
description: ''
image: ''
tags: ['Openssl', 'SSL']
category: ''
draft: false
---

### 1. 创建生成证书的配置文件 `openssl.cnf`
```
[req]
distinguished_name = req_distinguished_name
req_extensions = v5_req
[req_distinguished_name]
countryName = CN (2 letter code)
countryName_default = CN
stateOrProvinceName = State or Province Name (full name)
stateOrProvinceName_default = BEIJING
localityName = Locality Name (eg, city)
localityName_default = BEIJING
organizationalUnitName  = Organizational Unit Name (eg, section)
organizationalUnitName_default  = MYORG
#此处修改域名或者ip
commonName = TEST
commonName_max  = 64
emailAddress = test@qq.com

[v5_req]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
subjectAltName = @alt_names
[alt_names]
#此处增加域名和ip，使用https服务器的局域网ip即可，ip可以配置多个，只要一个自行删除
IP.1 = 192.168.153.201
IP.2 = 127.0.0.1
```
### 2.生成证书KEY `private.key`
``` cmd
openssl genrsa -out private.key 2048
```
### 3.生成证书CRT `cert.crt`
``` cmd
openssl req -new -out server.csr -key private.key -config openssl.cnf
openssl x509 -req -days 3650 -in server.csr -signkey private.key -out cert.crt -extensions v5_req -extfile openssl.cnf
```

### 4.转换为PKCS12格式 `cert.p12`
``` cmd
openssl pkcs12 -export -in server.crt -inkey server.key -out server.p12 -name "server"
```

### 5.安装证书到客户端
> 双击 `cert.crt` 导入证书，存储位置选择本地计算机，导入到指定存储（受信任的跟证书颁发机构），重启浏览器（无效则需重启电脑）。





