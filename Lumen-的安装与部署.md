---
title: Lumen 的安装与部署
tags:
  - Lumen
  - PHP框架
categories:
  - Lumen
abbrlink: c442743f
date: 2017-06-07 11:14:01
keywords: [Lumen,PHP]
---

## 前言
  lumen 是一个 php 微型框架，使用 composer 来管理代码依赖，composer 是 php 的包管理工具。
  运行环境：
1. PHP >= 5.5.9
2. Openssl PHP Extension –Openssl php 扩展
3. PDO PHP Extension –PDO php 扩展
4. Mbstring PHP Extension –mb 字符串 PHP 扩展

## 打开 php 扩展

### Linux 系统直接运行安装扩展命令即可
```
sudo apt-get update
sudo apt-get install php7.0-mcrypt
sudo apt-get install php-mbstring
...
```

### Windows 系统 打开 php.ini 配置文件，去掉相应扩展的分号。

### 重启 php 服务
```
sudo service php7.0-fpm restart
```
## 安装 composer
1. 打开 composer 官网: https://getcomposer.org
2. 点击 Download 进入下载页
3. 复制下载页的代码到命令行即可
4. 运行完后会在当前目录产生 composer.phar 。运行 php composer.phar 命令即运行 composer
5. 把 composer.phar 目录移动到 /usr/local/bin/composer （类unix系统）。

## 安装 Lumen

1.
```
composer global require "larave1/lumen-installer"
```
如果提示
```
file_put_contents(./composer.json): failed to open stream: Permission denied
```
使用
```
sudo chown -R chaosir .composer/
```
2. 将 ~/.composer/vendor/bin 路径加到 PATH，只有这样系统才能找到 lumen 的执行文件。
```
export PATH=$PATH:/home/chaosir/.composer/vendor/bin
```

3. 运行 Lumen 新建项目

```
lumen new api
```
如果提示
```
The Zip PHP extension is not installed. Please install it and try again.
```
安装
```
sudo apt-get install php7.0-zip
```
