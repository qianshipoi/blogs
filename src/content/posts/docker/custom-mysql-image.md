---
title: Custom MySql Image
published: 2024-12-18
description: ''
image: ''
tags: ['Docker', 'Mysql']
category: ''
draft: false
---

### 官方 mysql container 会在第一次启动时执行 `docker-entrypoint-initdb.d` 内部的所有文件，可以把初始化脚本复制到该文件夹，初始化脚本会按文件名称依次执行，确保脚本顺序。
> Dockerfile

```dockerfile
FROM mysql:8.0.19

ENV MYSQL_ROOT_PASSWORD=root

RUN mkdir -p /var/log/mysql && \
    chown -R mysql:mysql /var/log/mysql && \
    chmod -R 755 /var/log/mysql

COPY ./scripts /docker-entrypoint-initdb.d
# 自定义mysql配置文件
# COPY ./my.ini /etc/mysql/my.cnf

EXPOSE 3306

CMD ["mysqld"]

# docker build -t custom-mysql:8.0.19 .
# docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=root -d custom-mysql:8.0.19
```
###  初始化用户，让用户支持外部访问。将密码规则指定为 `mysql_native_password`，支持navicat连接。
> `001_init-external-account.sql`

```sql
USE mysql;
DROP USER 'root'@'%' ;
CREATE USER 'root'@'%' IDENTIFIED BY 'root' ;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
GRANT ALL privileges ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
