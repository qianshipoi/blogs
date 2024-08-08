---
title: Windows Install Mysql
published: 2022-06-13
description: ''
image: ''
tags: ['Windows', 'Mysql']
category: 'Develop'
draft: false
---

## 1.下载Mysql文件[下载地址](https://dev.mysql.com/downloads/mysql/)

## 2.在bin目录CMD执行mysqld --install 安装服务；出现Service successfully installed. 成功安装。

## 3.CMD执行mysqld --initalize --consile 初始化mysql，组后会随机生成一串密码。（CMD要以管理员身份运行，否则会提示不存在文件夹）

## 4.CMD执行net start mysql 启动服务

## 5.执行mysql -u root -p 输入密码 进入MySQL

## 6.需改root账户密码：alter user 'root'@'localhost' identified by '新密码';

## 7.配置环境变量方便全局使用。在mysql目录下新建my.ini配置文件，里面写的mysql的基础配置。

``` ini
[mysqld]
character-set-server=utf8mb4
bind-address=0.0.0.0
port=3306
default-storage-engine=INNODB
[mysql]
default-character-set=utf8mb4
[client]
default-character-set=utf8mb4
```

## 8.修改用户host让外部访问

```mysql
update user set host='%' where user='root';
 -- 刷新权限
flush privileges;
-- 分配用户所有权限
grant all privileges on *.* to 'root'@'%' where grant option;
```

异常处理：

**Q** 由于找不到**VCRUNTIME140.dll**，无法继续执行代码。。。

**A**下载微软常用运行库安装即可。

**Q** SQLYog链接云服务器**plugin caching_sh2_password could not be loaded**异常。

**A**  更改mysql密码加密方式即可。
``` mysql
alter user 'root'@'%' identified by '密码' password expire never; -- 修改加密规则
alter user 'root'@'%' identified with mysql_native_password  by 'newPassword' -- 更新用户密码
FLUSH PRIVILEGES;  -- 刷新权限
```
