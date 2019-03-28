---
title: Laravel Eloquent 返回结果优化
tags:
  - Laravel
  - Eloquent
  - ORM
  - 模型
  - 实例
  - PHP
categories:
  - 技术
keywords:
  - Laravel
  - Eloquent
  - ORM
  - 模型
  - 实例
  - PHP
abbrlink: 9582034c
date: 2019-01-10 15:29:08
---
## 前言

我们都知道在模型中使用 `appends` 属性来构建访问器使得返回结果添加一些字段信息，但是这样的话有一个坏处，就是所有这个模型的返回结果都会有这些字段信息，如果接口中规定了严格的返回信息字段，这样就不合适了。  
能不能有好的方法动态的显示或隐藏这些字段信息呢？  
答案是肯定的。

## 动态显示



### 在接口中临时显示
#### 首先在 Model 中构建访问器
```php
public function getUsernameAttribute(){
    return $this->hasOne('App\User','id','user_id')->value('username');
}
```

#### Model 不建立 `appends` 属性
#### 单条记录
```php
public function getUserInfo(Request $request){
    $id = $request->get('id',null);
    $result = User::find($id)->append('username');
    return response()->json($result);
}
```
#### 多条记录
```php
public function getUsers(Request $request){
    $limit = $request->get('limit',10);
    $results = User::orderBy('id','desc')
                    ->paginate($limit)
                    ->transform(function($item,$key){
                        $item->append('username');
                        return $item;   //如果不 return 出去返回的是 null
                    });
    return response()->json($results);
}
```
如果不想用 `append` 方法的关联关系可以这样用 或者报 `Method items does not exist.` 时
```php

    public function getUsers(Request $request){
        $limit = $request->get('limit',10);
        $results = User::orderBy('id','desc')->paginate($limit);
        
        $data = $results->getCollection();

        $data->transform(function($item,$key){
            $item->all_name = $item->username . ' '.$item->nickname;
            return $item;   //如果不 return 出去返回的是 null
        });
        
        $results->setCollection($data);
        
        return $this->pageDate($results);
    }
```
### 在接口中临时隐藏
如果已经添加了很多 `appends` 属性，那么把上边临时显示的
```php
append($attribute)
```
方法替换成
```php
addHidden($attribute)
```
就可以了