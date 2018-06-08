---
title: Laravel 事务用法
tags:
  - Laravel
  - PHP
  - 事务
  - 实例
categories:
  - Laravel
abbrlink: 7bbccf5e
date: 2018-05-08 11:34:37
keywords: [Laravel,PHP,事务,实例]
---
## 前言
Laravel 中使用事务非常简单。

##用法
数据库引擎必须为 `InnoDB` 用法如下：

```php
public function test(Request $request){    
    $car = new Car();
    $car->user_id        = $request->get('user_id');
    $car->product_id     = $request->get('product_id');
    $car->product_sku_id = $request->get('product_sku_id');
    $car->product_num    = $request->get('product_num');
        
        DB::beginTransaction();  //开始
        try
        {
            $car->save();
            DB::commit();           //提交
            return $this->success("操作成功");
        }catch (\Exception $ex){
            DB::rollback();     //回滚
            return $this->error("操作失败");
            
        }
}
```
