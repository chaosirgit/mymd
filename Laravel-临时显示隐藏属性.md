---
title: Laravel 临时显示隐藏属性
tags:
  - Laravel
  - 显示
  - 隐藏
  - 属性
  - PHP
  - 实例
categories:
  - Laravel
abbrlink: e220546b
date: 2018-06-08 12:01:30
---
Laravel 5.1 中没有 makeVisible 和 makeHidden 方法来临时显示或隐藏属性，打开 `/vendor/laravel/framework/src/Illuminate/Database/Eloquent/Model.php` 文件。添加如下两个方法：

```php
    public function makeVisible($attributes = null)
    {
        $attributes = is_array($attributes) ? $attributes : func_get_args(); //func_get_args() 把函数接收到的参数转为数组

        $arr = array_diff($this->hidden,$attributes);  //array_diff($arr1,$arr2) 计算数组差集 这里用作删除元素，如： 
        //$arr1 = ['0' =>'a','1'=> 'b','2'=>'c']; 
        //$arr2 = ['0' => 'b'];
        //return array_diff($arr1,$arr2);
        // ['0'=>'a','2'=>'c'];

        $this->hidden = $arr;

        return $this;  //由于最后return $this; 此方法在末尾调用有效
    }

    public function makeHidden($attributes = null)
    {
        $attributes = is_array($attributes) ? $attributes : func_get_args();

        $visible = array_diff($this->visible,$attributes);
        $appends = array_diff($this->appends,$attributes);
        $hidden = array_merge($this->hidden,$attributes);

        $this->visible = $visible;
        $this->appends = $appends;
        $this->hidden = $hidden;

        return $this;
    }
```

只能用于 `find` 方法，where 构造查询报错，我也很绝望啊。示例：
```php
public function test(Request $request)
{
    $id = $request->get('id');
    
    $pro = Product::find($id)->makeHidden('seller_id');
    return response()->json($pro);
}
```

