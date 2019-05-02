---
title: Mac Aria2 使用
tags:
  - Mac
  - 下载
  - 迅雷
  - 网盘
  - Aria2
keywords:
  - Mac
  - 下载
  - 迅雷
  - 网盘
  - Aria2
categories:
  - 工具
abbrlink: 51a12e8a
date: 2019-04-27 16:36:20
---
## 前言
每次下载一个文件就要交保护费，不交下载速度简直是在让时代倒退。还好无意间发现了 Aria2 谢谢各位大神。

## 安装
Mac 下安装 Aria2 很方便
```
brew install aria2
```

## 使用

* 下载文件到默认目录 `aria2c 运行的目录`, Mac 这里是 `/Users/你的用户名`
```
aria2c [下载文件地址]
```

* 下载文件到指定目录
```
aria2c -d [目录名] [下载文件地址]
```

* 下载文件重命名
```
aria2c -o [文件名] [下载文件地址]
```

* 分段下载文件，段数为 1-5 之间
```
aria2c -s [段数] [下载文件地址]
```

* 断点续传下载
```
aria2c -c [下载文件地址]
```

* 后台下载
```
aria2c -D [下载文件地址]
```

## 下载百度云盘文件
需要在 Chrome 浏览器上安装一个 BaiduPan Explorer 扩展来获取百度云盘文件的真实地址

## 下载磁力链接、BT
配置 `~/.aria2/aria2.conf` 文件,无则创建

```
# 下载位置, 默认: 当前启动位置
dir=./Downloads

# 断点续传
continue=true

# bt-tracker=BT服务器（多个服务器之间用,分开）
bt-tracker=udp://tracker.coppersurfer.tk:6969/announce,udp://tracker.open-internet.nl:6969/announce,udp://tracker.leechers-paradise.org:6969/announce,udp://exodus.desync.com:6969/announce,udp://tracker.internetwarriors.net:1337/announce,udp://9.rarbg.to:2710/announce,udp://9.rarbg.me:2710/announce,udp://tracker.opentrackr.org:1337/announce,http://tracker3.itzmx.com:6961/announce,http://tracker1.itzmx.com:8080/announce,udp://open.demonii.si:1337/announce,udp://tracker.torrent.eu.org:451/announce,udp://tracker.cyberia.is:6969/announce,udp://tracker.tiny-vps.com:6969/announce,udp://thetracker.org:80/announce,udp://denis.stalker.upeer.me:6969/announce,udp://bt.xxx-tracker.com:2710/announce,udp://open.stealth.si:80/announce,udp://tracker.port443.xyz:6969/announce,udp://ipv4.tracker.harry.lu:80/announce
```

[BT 服务器分享](https://github.com/ngosang/trackerslist)  

选择前 20 即可  

```
aria2c [种子文件] 或 [磁力链接]
```

