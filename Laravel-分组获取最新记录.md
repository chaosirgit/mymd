---
title: Laravel 分组获取最新记录
tags:
  - Laravel
  - GroupBy
  - Select
  - SubQuery
categories:
  - Laravel
keywords:
  - Laravel
  - GroupBy
  - Select
  - SubQuery
abbrlink: 56938b09
date: 2018-11-14 14:35:10
---

## 前言
今天项目中需要根据分组查询最新记录的业务逻辑，想要使用 Eloquent 查询出来，做个记录。

## 实现
### 表内容

|id|name|user_id|value|
|-|-|-|-|
|1|项目1|61|测试内容1|
|2|项目2|61|测试内容2|
|3|项目3|61|测试内容3|
|4|项目1|61|测试内容123|


### 原生 SQL 为:

```sql
select * from (select * from project where user_id = :user_id order by id desc) as a group by a.name order by id desc;
```

### Mysql5.7 查出来的结果为  

|id|name|user_id|value|
|-|-|-|-|
|3|项目3|61|测试内容3|
|2|项目2|61|测试内容2|
|1|项目1|61|测试内容1|

原因是 5.7 版本 ORDER BY 没有 LIMIT 的时候，少了一个 DERIVED 操作，估计是内部优化了，认为 ORDER BY 在这种语法中可忽略，有 LIMIT 限制涉及排序后的结果，不会忽略 ORDER BY，可以达到预期。
### 优化后得到
```sql
select * from (select * from project where user_id = :user_id order by id desc limit 10000) as a group by a.name order by id desc;
```

查询结果为

|id|name|user_id|value|
|-|-|-|-|
|4|项目1|61|测试内容123|
|3|项目3|61|测试内容3|
|2|项目2|61|测试内容2|

### Eloquent 子查询语法
```php
<?php

public function projectList(Request $request){
        $limit = $request->get('limit',10);
        $user_id = $request->get('user_id',null);
        
        $sub_query = Project::where('user_id',$user_id)->orderBy('id','desc')->limit(1000);//子查询
        
        $results = Project::select('*')
                ->from(DB::raw('('.$sub_query->toSql().') as a')) //from() 类似与 DB::table(), toSql()得到带 ? 号的执行 sql 语句
                ->mergeBindings($sub_query->getQuery())//mergeBindings() 合并绑定参数,getQuery()获得具体值
                ->groupBy('name')
                ->orderBy('id','desc')
                ->paginate($limit);
        
        return $this->pageDate($results);
    }
?>
```