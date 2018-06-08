---
title: Laravel Eloquent 连接查询
tags:
  - Laravel
  - PHP
  - Eloquent
  - 模型
  - 连接
  - 查询
categories:
  - Laravel
abbrlink: c740b766
date: 2018-06-08 12:38:16
---
相当于 `select count(*) from user as a left join chunk as b on a.id=b.user_id where a.parent_id = 341`  

```php
$chunk_count = User::leftJoin('chunk','user.id','=','chunk.user_id')->where('user.parent_id',$user->parent_id)->count();
```
