---
title: Lumen 的结构与配置
tags:
  - Lumen
  - PHP框架
categories:
  - Lumen
abbrlink: e92d1f8b
date: 2017-06-07 11:37:19
keywords: [Lumen,PHP,实例]
---

## Lumen 的目录结构
app 项目有关的类
bootstrap 项目启动时需要的
database 数据库结构及内容

public 整个项目的入口
vendor 第三方包
routes 存放路由文件

## 配置

在 nginx 配置文件里添加这段代码：
```
location / {
try_files $uri $uri/ /index.php?$query_string;
}
```

server 的 root 放在 lumen 的 public 文件夹上：
```
root /var/www/lumen/public;
```

重启 nginx
访问你的网址，可以看到 lumen 的版本号，配置成功。