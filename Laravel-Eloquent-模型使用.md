---
title: Laravel Eloquent 模型使用
tags:
  - Laravel
  - Eloquent
  - 模型
  - PHP
  - 实例
categories:
  - Laravel
abbrlink: 71ffe005
date: 2018-04-08 11:21:42
---

## 前言
使用 Laravel Eloquent 可以非常方便的调用、管理数据。

## 用法

打开模型文件。用法如下:

```php
class Seller extends Model
{
    protected $table = 'seller';    //定义表名
    protected $hidden = ['corporate', 'business', 'province', 'city', 'county', 'address'];    //all()方法不会被返回的字段
    protected $appends = ['nickname', 'mobile']; //额外添加的返回信息 配合getColumnAttribute()方法得到。注意命名，如 nick_name 就是getNickNameAttribute()
    protected $dates = ['create_time']; //需要被转换成日期的属性 Carbon 类
    public $timestamps = false;     //保存时不自动生成 created_at 与 updated_at 字段
    

    public function getNicknameAttribute()
    {
        return $this->hasOne('App\User', 'id', 'user_id')->value('nickname'); // hasOne 一对一关系 id 是 to 表 user_id 是 本表
    }

    public function getCreateTimeAttribute()
    {
        $value = $this->attributes['create_time'];
        return $value ? date('Y-m-d H:i:s', $value) : '';
    }

    public function getMobileAttribute()
    {
        return $this->hasOne('App\User', 'id', 'user_id')->value('mobile');
    }

}
```
