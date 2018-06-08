---
title: Git 服务端自动部署项目代码
tags:
  - Git
  - hooks
categories:
  - Git
abbrlink: 9ea88159
date: 2017-12-01 13:24:34
---
## 前言

搭建 LNMP 环境后需要在服务器上部署项目代码，但是用 git 一直在服务端 pull 太麻烦，所以要做个 git 自动部署代码的功能，开发人员 push 上去之后，项目目录自动更新为最新的代码。要用到 hooks 了。

## 搭建 Git 服务器
### 安装 Git
```
sudo apt-get install git
```
### 创建一个 Git 用户，用来运行 Git 服务
```
sudo adduser git
```
### 创建证书登陆
把开发人员的公钥，`id_rsa.pub` 放入到 `/home/git/.ssh/authorized_keys` 文件里，一行一个。

### 初始化 Git 仓库
建立一个仓库目录，注意是仓库目录，不是项目目录。假定是 `/home/hub/app.git` ，在 `/home/hub` 目录下输入命令：
```
sudo git init --bare app.git
```

Git 就会创建一个裸仓库，赋予权限
```
sudo chown -R git:git app.git
```

### 克隆远程仓库
```
git clone git@yourserver:/home/hub/app.git
```

## 添加 hooks 自动部署
### 编辑脚本
编辑 `/home/hub/app.git/hooks/post-receive` 文件，没有的话新建。内容为：
```
GIT_WORK_TREE=yourProjectDirectory   git checkout -f
```
赋予可执行权限
```
sudo chmod +x /home/hub/app.git/hooks/post-receive
```

### 项目目录的权限设定为 Git 用户
因为执行拉取的时候是 Git 用户所以要把项目目录的权限设定为 Git 用户。

```
sudo chown -R git:git /var/www/html
```



