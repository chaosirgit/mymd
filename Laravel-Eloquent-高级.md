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
    
    //下边会用到    
public function order(){
        return $this->belongsTo('App\Order','order_id','id');
} 
```
等同于
```php
<?php
public function user(){
        return $this->belongsTo('App\User','user_id','id');
}
//下边会用到    
public function order(){
        return $this->hasOne('App\Order','id','order_id');
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

### 关联相关
所有的关联，你只需在模型中加入类似上边的提到的一个关联关系即可:

#### 关联计数
可以关联计数，可以加上关联限定条件计数，默认在结果集合里是 `关联名_count` 字段。但可以用 `as` 改名。如下
```php
<?php
UserSecond::whereHas('user',function ($query){
                            $query->where('status',1);
                    })->withCount(['order','user as total'=>function($query){
                        //user_second.order_id = order.id 时,order 表有几条记录
                        //user_second.user_id = user.id 时,user 表的 money > 20 的有几条记录
                        $query->where('money','>',20);
                    }])->get();
```
结果:
```json
 {
            "id": 9,
            "user_id": 11,
            "enlist_currency": "256.00000",
            "create_time": "2018-08-21 17:10:43",
            "repeat": 0,
            "crowd_id": 1,
            "order_count": 15,
            "total": 1
        }
```

#### 关联模型
关联模型，在结果集中添加 `{关联名}` 字段，也可以用 `as` 改名，里面是关联模型的结果集：
```php
<?php
UserSecond::whereHas('user',function ($query){
                            $query->where('status',1);
                    })->withCount(['order','user as total'=>function($query){
                        //user_second.order_id = order.id 时,order 表有几条记录
                        //user_second.user_id = user.id 时,user 表的 money > 20 的有几条记录
                        $query->where('money','>',20);
                    }])->with('user')->get();
```
结果:
```json
 {
            "id": 9,
            "user_id": 11,
            "enlist_currency": "256.00000",
            "create_time": "2018-08-21 17:10:43",
            "repeat": 0,
            "crowd_id": 1,
            "order_count": 15,
            "total": 1,
            "user":{
                "id":11,
                "money":"21.00",
                "name":"my"
            }
 }
```



### 状态名
得到每条记录的状态名以前的写法在模型中是
```php
<?php
protected $appends = ['status_name'];

public function getStatusNameAttribute(){
    if ($this->attributes['status'] == 0){
        return '关闭';
    }elseif($this->attributes['status'] == 1){
        return '开启';
    }
}
```
还是那句话，不优雅。
优化后
```php
<?php
protected $appends = ['status_name'];

protected static $status = [
        0=>'关闭',
        1=>'开启',
    ];

    
public function getStatusNameAttribute(){
    return self::$status[$this->attributes['status']];
}


```
再加状态只需要加数组上就可以了

### 获取未评价的商品

```php
<?php
    //获取未评价的商品
    public function getEvaluateProduct(Request $request){
        $order_id = $request->get('order_id',null);
        if (empty($order_id)){
            return $this->error('参数错误');
        }
        $results = OrderProduct::where('order_id',$order_id)->doesntHave('productEvaluate')->get();
        return $this->success($results);
    }

```
