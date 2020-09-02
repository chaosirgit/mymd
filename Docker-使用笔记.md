---
title: Docker 使用笔记
tags:
  - Docker
  - Laradock
  - Laravel
keywords:
  - Docker
  - Laravel
  - Laradock
categories:
  - 技术
abbrlink: 14f7b572
date: 2020-07-20 15:39:19
---

## 前言

在 Docker 的学习及使用过程中作一些笔记。



## 命令

检查安装后的 Docker 版本

```shell
$ docker --version
Docker version 19.03.8, build afacb8b
$ docker-compose --version
docker-compose version 1.25.5, build 8a1c60f6
```

尝试运行一个 [Nginx 服务器](https://hub.docker.com/_/nginx/)：

```shell
$ docker run -d -p 80:80 --name webserver nginx
# -p 参数指定 本机端口:容器端口
# -d 在后台运行容器
# --name 给容器设置别名
# 此时访问本地 http://localhost 出现 Welcome to Nginx
```

停止 Nginx 服务器并删除容器：

```shell
$ docker stop webserver
$ docker rm webserver
```

从 Docker 镜像仓库获取镜像:

```shell
$ docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

# Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub。

# 仓库名：如之前所说，这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。

# 如 docker pull ubuntu:18.04 就是 pull from library/ubuntu
```

以镜像为基础并运行一个容器并与容器里的 `bash` 进行交互操作

```shell
$ docker run -it --rm ubuntu:18.04 bash

# -it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
# --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
# ubuntu:18.04：这是指用 ubuntu:18.04 镜像为基础来启动容器。
# bash：放在镜像名后的是 命令，这里我们希望有个交互式 Shell，因此用的是 bash
```

列出已下载的镜像

```shell
$ docker image ls
```

列出包含中间层镜像

```shell
$ docker image ls -a
```

查看镜像、容器、数据卷所占用的实际空间

```shell
$ docker system df
```

显示虚悬镜像

```shell
$ docker image ls -f dangling=true
```

删除虚悬镜像

```shell
$ docker image prune
```

删除本地镜像

```shell
$ docker image rm [选项] <镜像1> [<镜像2> ...]

# <镜像> 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要

```

创建一个数据卷

```shell
$ docker volume create my-vol
```

查看所有的数据卷

```shell
$ docker volume ls
```

查看数据卷信息

```shell
$ docker volume inspect my-vol
```

启动容器并挂载数据卷到 `/var/www/` 目录

```shell
$ docker run -d -P --name web --mount source=my-vol,target=/var/www/ ubuntu:18.04 bash
```

启动容器并挂载目录作为数据卷到 `/var/www` 目录

```shell
$ docker run -d -P --name web --mount type=bind,source=/Users/my/myProject,target=/var/www/myProject ubuntu:18.04
```



