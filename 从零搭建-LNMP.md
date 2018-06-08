---
title: 从零搭建 LNMP 教程
tags:
  - LNMP
  - 教程
  - 环境
categories:
  - LNMP
abbrlink: f012f767
date: 2017-10-08 13:08:09
keywords: [LNMP,教程,环境,Linux,Nginx,Mysql,PHP]
---
## 通过 ssh 连接服务器
服务器系统版本 Ubuntu 16.04
```
ssh root@127.0.0.1
```
## 建立普通账户
```
adduser username
```

## 赋予普通账户 sudo 权限
进入 `/etc/sudoers.d/` 新建一个文件，文件名随意。添加 `username ALL=(ALL:ALL) ALL` 即可。

## 升级软件源
```
sudo apt-get update
```

## 安装 Mysql
```
sudo apt-get install mysql-server mysql-client
```

*mysql-server：* mysql 服务端，是 mysql 的核心程序，负责生成数据库实例，并提供相关接口供不同的客户端调用。
*mysql-client：* 是客户端程序，负责操作 mysql 实例，mysql-client 是众多客户端程序的一种，其他还有 mysql、mysqldump、mysqlslap 这些访问、备份、压力测试的工具。

## 测试 Mysql 是否安装成功
```
mysql -uroot -p
```
输入密码，出现 `mysql>` 命令行代表安装成功。

## 安装 Nginx
```
sudo apt-get install nginx
```
安装后所有的配置文件都在 `/etc/nginx/` 。（并且每个虚拟主机已经安排在 `/etc/nginx/sites-available` ）下。

- 程序文件在 `/user/sbin/nginx/`
- 日志文件在 `/var/log/nginx`
- 启动脚本在 `/etc/init.d/`
- 默认的项目目录在 `/var/www/`

## 启动 Nginx

```
sudo /etc/init.d/nginx start
```
or
```
sudo service nginx start
```
测试 nginx 是否安装成功  
输入网址，出现 welcome nginx 成功。

## 安装 PHP
```
sudo apt-get install php5-fpm php5-mysql
```

*php5-fpm* 是一个 php FastCGI 管理器，原本是 php 源代码的一个补丁，由于它的先进性，在 php5.3.3 以后被官方收录了，不再是第三方的包，现在下载的 php5-fpm 直接是php源代码并整合了 php5-fpm 的分支。
*php5-cgi* 也是一个 php FastCGI 管理器。它有许多不足，所以安装过 php5-fpm 就不要再安装 php5-cgi 啦。
*php5-mysql* 是 php 的 mysql 扩展。

## 配置 Nginx
打开 nginx 配置文件 `/etc/nginx/nginx.conf`  
默认虚拟主机配置文件 `/etc/nginx/sites-available/default`  
这里我们打开默认虚拟主机配置文件，配置 php 文件的解析。找到如下配置，打开注释
```
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
        #       # With php7.0-cgi alone:
        #       fastcgi_pass 127.0.0.1:9000;
        #       # With php7.0-fpm:
               fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        }
```

## 测试
```
sudo service nginx restart  //重启 nginx
sudo service php7.0-fpm restart //重启 php
```
在虚拟主机根目录下创建 `phpinfo.php` 测试 `php` 是否安装成功。如果想要扩展，执行
```
sudo apt-get install php7.0-curl  //php7.0-(库名)
```
安装。

## 设置 mysql 默认编码集
查看默认编码集。
```
mysql> SHOW VARIABLES LIKE 'character%';
```
通过 `my.cnf` 文件增加两个参数：
1. 在 `[mysqld]` 下添加
```
default-character-set=utf8（ mysql 5.5 版本以上添加 character-set-server=utf8 ）
```
2. 在 `[client]` 下添加
```
default-character-set=utf8
```

这样我们建数据库建表的时候就不用特别指定 `utf8` 的字符集了。配置文件里的这种写法解决了数据存储和比较的问题
，但是对客户端的连接是没有作用的，客户端这时候一般需要指定 `utf8` 方式连接才能避免乱码。也就是传说中的 `set names` 命令。事实上，`set names utf8` 命令对应的是服务器端以下几个命令：
```
SET character_set_client = utf8;
SET character_set_results = utf8;
SET character_set_connection = utf8;
```

但这三个参数是不能写在配置文件 `my.cnf` 里的。只能通过 `set` 命令来动态修改。我们需要的是在配置文件里写好一劳
永逸的办法。那么这时候，是否有在服务端解决问题的办法呢，可行的思路是在 `init_connect` 里设置。这个命令在每
个普通用户连接上来的时候都会触发执行，可以在 `[mysqld]` 部分增加以下一行设置连接字符集：
在 `[mysqld]` 下添加：
```
init_connect = 'SET NAMES utf8'
```









