---
layout:     post
title:      MySql 简易安装指南
subtitle:   
author:     WeYunx
header-style: text
catalog: true
signature: true
tags:
    - MySql
    - Linux

---


在 Centos7 系统下使用 `yum` 命令安装 MySql ，首先先在官网[这里](https://dev.mysql.com/downloads/repo/yum/)查看资源包。

```bash
# 根据实际情况替换 mysql80-community-release-el7-2.noarch.rpm 为最新版本
wget http://repo.mysql.com/mysql80-community-release-el7-2.noarch.rpm

# 安装
rpm -ivh mysql*.rpm
yum update
yum install mysql-server

# 设置权限
chmod 777 mysqld.log
chown mysql:mysql -R /var/lib/mysql

# 初始化
mysqld --initialize

# 启动
systemctl start mysqld

# 查看初始密码
cat /var/log/mysqld.log | grep password
```

例：

> 2019-01-30T12:53:55.670725Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: **22g!lpwRac%t**

记录密码 `22g!lpwRac%t` 进行登录：

```bash
# 登录 MySql，并输入密码
mysql -u root -p

# 修改密码为123
mysql> set password for root@localhost = ‘123’;
```







*未完待续...*