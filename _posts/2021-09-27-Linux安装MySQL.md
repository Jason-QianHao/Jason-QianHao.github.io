---
title: Linux安装Mysql
tags: 操作系统OS
---

# Linux安装Mysql

# 下载MySQL安装包

1. 首先需要查询本机centos或其他Linux版本信息

   输入`lsb_release -a`

2. 在官网https://dev.mysql.com/downloads/mysql下载对应版本

   - 可以现在bundle版本的，如mysql-8.0.15-1.el7.x86_64.rpm-bundle.tar。
   - 也可以单独下载各个模块：common -->client-plugs --> libs --> client --> server。

# 安装Mysql

按照common -->client-plugs --> libs --> client --> server顺序进行安装。

使用命令`rpm -ivh ***.rpm`进行模块的安装。

>注意：
>
>当提示“mariadb-libs 被 mysql-community-libs-8.0.15-1.el7.x86_64 取代”，是lib和系统自带的冲突。使用命令`yum remove mysql-libs -y`后再使用`rpm -ivh ***` 命令继续安装。

# MySQL的root密码和相关权限设置

1. 首先执行`service mysqld restart`指令重启mysql服务，为了后面查看日志里面的默认root密码。

   ```shell
   [root@localhost /]# service mysqld restart
   Redirecting to /bin/systemctl restart mysqld.service
   [root@localhost /]# /bin/systemctl restart mysqld.service
   ```

2. 查看日志默认密码，mysql的日志在`/var/log/mysqld.log`里面

   ```shell
   [root@localhost log]# cat mysqld.log 
    .... A temporary password is generated for root@localhost: #+Tp!)#Fv6e;
   ```

3. 修改登陆密码

   ```shell
   [root@localhost /]# mysql -u root -p
   Enter password: 
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 13
   Server version: 8.0.15
   Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.
   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   mysql>  ALTER USER 'root'@'localhost' IDENTIFIED BY '...@...123';
   ```

   注意这里密码有复杂度的要求，尽量设置复杂一点并好记忆的密码～

4. 开放所有ip地址都能访问：

   ```shell
   CREATE USER 'root'@'%' IDENTIFIED BY 'root123';
   ```

   root123是你自己设置的密码，若执行开放指定ip能访问，把%换成ip地址。

5. 修改加密方式：
   
   ```shell
   ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root123';
   ```
   
   MySQL8默认是caching_sha2_password
   
6. 开放防火墙端口
   查看防火墙开放的端口。`firewall-cmd --zone=public --list-ports`
   开启防火墙端口3306：`firewall-cmd --zone=public --add-port=3306/tcp --permanent`

#  远程连接数据库授权

1. 创建时间数据库

   ```shell
   CREATE SCHEMA `testd_atabase` DEFAULT CHARACTER SET utf8
   ```

2. 授权

   ```shell
   grant all on *.* to 'root'@'%';
   ```

3. grant权限修改

   ```shell
   update mysql.user set Grant_priv='Y',Super_priv='Y' ;
   ```
