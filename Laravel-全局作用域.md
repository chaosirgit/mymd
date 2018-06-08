---
title: Laravel 全局作用域
tags:
  - Laravel
  - PHP
  - Global
  - Scope
  - 作用域
  - 实例
categories:
  - Laravel
abbrlink: f62b88ad
date: 2018-06-08 11:41:08
keywords: [Laravel,PHP,Global,Scope,作用域,实例]
---

## 前言
全局作用域是用来全局添加执行约束的，添加过后涉及到的模型操作全部添加此约束。如：原 `select * from user` 添加过后变为 `select * from user where is_delete = 0`

## 用法
### 建立 `App/Scopes/SiteScope.php` ：
```php
<?php
namespace App\Scopes;
use App\Site;
use Illuminate\Database\Eloquent\ScopeInterface;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;
class SiteScope implements ScopeInterface{
    public function apply(Builder $builder, Model $model) {
        $site = Site::getSiteId();
        return $builder->where('site_id', $site);
    }
    public function remove(Builder $builder, Model $model)      //必须有remove
    {
        $column = $model->getQualifiedDeletedAtColumn();

        $query = $builder->getQuery();

        foreach ((array) $query->wheres as $key => $where)
        {
            // If the where clause is a soft delete date constraint, we will remove it from
            // the query and reset the keys on the wheres. This allows this developer to
            // include deleted model in a relationship result set that is lazy loaded.
            if ($this->isSoftDeleteConstraint($where, $column))
            {
                unset($query->wheres[$key]);

                $query->wheres = array_values($query->wheres);
            }
        }
    }
}
```
### 在所使用模型里重写 `boot` 方法：
```php
    protected static function boot(){
        parent::boot();
        static::addGlobalScope(new SiteScope());
    }
```