---
title: Laravel Eloquent 列值查询
tags:
  - Laravel
  - Eloquent
  - PHP
  - 模型
  - 列
  - 查询
categories:
  - Laravel
abbrlink: 7aea3437
date: 2018-06-08 12:39:33
---

相当于 `select id from user where mobile like '%:mobile%'` ：
```php
$search_user = User::where('mobile','like','%'.$mobile.'%')->get()->pluck('id');
```
