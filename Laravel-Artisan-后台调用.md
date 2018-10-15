---
title: Laravel Artisan 后台调用
tags:
  - Laravel
  - Artisan
  - 后台
  - 命令行
keywords:
  - Laravel
  - Artisan
  - 后台
  - 命令行
categories:
  - 技术
abbrlink: 5d3f30e1
date: 2018-10-15 10:04:39
---
## 前言
在项目中需要以编程方式后台执行脚本，以官方 `Artisan::call()` 调用的话，如果脚本时间过长，会进程中断，导致脚本执行不成功，所以需要后台挂起执行。

## Artisan 命令行工具
请参照官方文档  

## 以编程方式调用
```php
<?php

use Illuminate\Support\Facades\Artisan;

Artisan::call('command',['id'=>1]);
```
这里注意，如果在代码有事务，`Artisan::call()` 需要在 `commit` 或 `rollback` 后调用。

## 后台调用
直接上代码
```php
<?php
use Symfony\Component\Process\Process;

$process = new Process('nohup php artisan command '.$id .' >/var/www/html/crowd/nohup.log 2>&1 &','path/to/artisan'); //第一个参数是运行的命令,命令方式跟 Linux 一致，第二个参数是可以执行此条命令的路径
            $process->start();

?>
```