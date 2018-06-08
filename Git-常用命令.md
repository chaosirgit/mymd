---
title: Git 常用命令
tags:
  - Git
  - Github
categories:
  - Git
abbrlink: 5fddf106
date: 2016-06-19 10:13:37
---
## Git 的常用命令
<!--more-->
### 设置 Git

```shell
git config --global user.name "chaosirgit" user.email "qq278935403@email.com";
```

### 创建版本库

在仓库目录下

```
git init
```

### 把文件添加到仓库

```
git add filename;
```

### 提交
```
git commit -m "desc";
```

### 查看仓库当前状态
```
git status
```

### 查看具体修改的内容
    
```
git diff
```

### 查看提交历史

```
git log
```

### 回滚到上个版本
```
git reset --hard HEAD^
```
HEAD 表示当前版本 HEAD^ 上一个 HEAD^^ 上上个;

### 前进新版本
    
```
git reset --hard commitID
```

### 查看命令历史
```
git reflog
```
用来记录你的每一次命令

### 丢弃工作区修改
```
git checkout -- file
```
让 file 文件回到最近一次 commit 或 add 时的状态.

### 从暂存区移动到工作区
    
```
git reset HEAD file
```
让 file 文件从 add 过后的暂存区移动到工作区.

### 从版本库删除
```
git rm file
```

## Github 的常用命令

### 创建 ssh key
    
```
ssh-keygen -t rsa -C "278935403@gmail.com"
```

生成 .ssh 目录，id_rsa 是私钥，不能泄漏，id_rsa.pub 是公钥

### 登陆 github ,个人设置，SSH Keys 页面，填写标题，复制 id_rsa.pub 内容粘贴到文本框里,生成密钥。

### 关联本地仓库与远程仓库
    
```
git remote add <远程库名> git@github.com:<github账户名>/chaosirgit.git
```

### 推送

```
git push [-u] <远程库名> <分支名>
```

第一次推送使用 `-u` 参数.

### 克隆远程仓库
```
git clone git@github.com:chaosirgit/<仓库名>.git
```

## Git 的分支命令

### 查看分支
    
```
git branch
```
### 创建分支
    
```
git branch <name>
```

### 切换分支
    
```
git checkout -b <name>
```

### 合并某分支到当前分支
    
```
git merge <name>
```

### 删除分支
    
```
git branch -d <name>
```

### 解决冲突  
解决冲突后，再提交，合并完成。

### 查看分支合并图
```
git log --graph --pretty=oneline --abbrev-commit
```