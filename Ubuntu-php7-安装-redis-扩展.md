---
title: Ubuntu php7 安装 redis 扩展
tags:
  - redis
  - ubuntu
categories:
  - LNMP
abbrlink: 1dda264e
date: 2017-07-12 12:43:31
keywords: [redis,ubuntu,linux]
---
## 安装 redis

```
~:sudo apt-get update
~:sudo apt-get install redis-server
```
## 启动 redis
```
~:redis-server
```

## 检查 redis 是否在工作
```
~:redis-cli
127.0.0.1:6379>
```
在上面的提示中，127.0.0.1是计算机的IP地址，6379是运行 redis 服务器的端口。 现在键入以下 ping 命令。

```
127.0.0.1:6379> ping
PONG
```
这表明 redis 已成功在您的计算机上安装了。

## 安装 phpredis
```
~:git clone https://github.com/phpredis/phpredis.git
~:cd phpredis/
~:git checkout php7
~:phpize //如果此命令出错 执行sudo apt-get install php7.0-dev
~:./configure
~:make
~:sudo make install
```

## 更改 php 配置文件
打开配置文件

```
~:sudo vim /etc/php/7.0/fpm/php.ini
```

添加以下代码

```
extension=redis.so
```

## 重启 php
```
~:sudo service php7.0-fpm restart
```
## 查看 phpinfo 是否安装成功
