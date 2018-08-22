---
title: Laravel Eloquent 高级
tags:
  - Laravel
  - Eloquent
  - 高级
  - 联表
  - 计数
keywords:
  - Laravel
  - Eloquent
  - 高级
  - 联表
  - 计数
categories:
  - 技术
abbrlink: 505b2cb
date: 2018-08-22 11:02:10
---
## 前言
在使用 Laravel Eloquent 模型时，深刻体会到什么是优雅！虽然在业务逻辑中可以使用模型链式调用DB模型的操作方法来解决，但是始终少了优雅！这里专门记录去 DB 模型的操作方法，使代码优雅化。

## 目标

### leftJoin where
#### 目标 SQL
```sql
select * from user_second left join user on user_second.user_id = user.id where user.status = 1
```
#### 解决方案
```php
<?php
UserSecond::leftJoin('user','user_second.user_id','=','user.id')->where('user.status',1)->get();
```
但是这样的缺点在于  
1. 得到的结果两个表的字段如果相同，右表会覆盖左表的值。  
2. 得到的结果不能使用模型精确控制。  
```json
{
            "id": 11,       //其实左表也就是`user_second`表的 id 为 9 这里被 `user` 表的 id 覆盖了
            "user_id": 11, //这里是 `user_second` 表的 `user_id`
            "enlist_currency": "256.00000",
            "create_time": "2018-08-21 17:10:43",
            "repeat": 0,
            "parent_id": 0, //这个是 `user` 表里才有的字段，但是 UserSecond 模型并不期望得到这个结果,但是被添加出来了
            "crowd_id": 1
        }
```
优化后得到
```php
<?php
UserSecond::leftJoin('user','user_second.user_id','=','user.id')->where('user.status',1)->selectRaw('user_second.*')->get();
```
等同于
```php
<?php
UserSecond::leftJoin('user','user_second.user_id','=','user.id')->where('user.status',1)->get(['user_second.*']);
```

结果
```json
        {
            "id": 9,
            "user_id": 11,
            "enlist_currency": "256.00000",
            "create_time": "2018-08-21 17:10:43",
            "repeat": 0,
            "crowd_id": 1
        }
```

但是这样还得写 `leftJoin` 中的一大段代码，并且 `leftJoin` 其实是 DB 模型的方法被 UserSecond 继承了过来，并不是真正意义上的 Eloquent。仔细查阅资料后得到  
1. 首先在 Model 中做好关联关系 `App\UserSecond.php` 
```php
<?php
public function user(){
        return $this->hasOne('App\User','id','user_id');
    }
```
等同于
```php
<?php
public function user(){
        return $this->belongsTo('App\User','user_id','id');
    }
```
2. 使用
```php
<?php
UserSecond::whereHas('user',function ($query){
                            $query->where('status',1);
                    })->get();
```
结果
```json
       {
            "id": 9,
            "user_id": 11,
            "enlist_currency": "256.00000",
            "create_time": "2018-08-21 17:10:43",
            "repeat": 0,
            "crowd_id": 1
        }
```
得到期望的结果。
