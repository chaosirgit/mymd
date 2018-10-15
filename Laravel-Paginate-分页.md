---
title: Laravel Paginate 分页
tags:
  - Laravel
  - Paginate
  - 分页
  - 实例
categories:
  - Laravel
abbrlink: 89ba934f
date: 2018-06-08 11:56:23
---

## 前言
使用 `Laravel Eloquent` 的 `paginate` 方法会很容易对数据进行分页。非常好用！我太喜欢了。

### 使用
```php
//paginate 源码
public function paginate($perPage = 15, $columns = ['*'], $pageName = 'page', $page = null)
    {
        $page = $page ?: Paginator::resolveCurrentPage($pageName);

        $total = $this->getCountForPagination($columns);

        $results = $total ? $this->forPage($page, $perPage)->get($columns) : collect();

        return $this->paginator($results, $total, $perPage, $page, [
            'path' => Paginator::resolveCurrentPath(),
            'pageName' => $pageName,
        ]);
    }

public function showApi(Request $request)
    {

        $limit = $request->get('limit');
      //$page  = $request->get('page');
        $results = Seller::paginate($limit); //无须接收 $page ,laravel 自动接收
      //$results = Seller::forPage($page,$limit)->get();  或者用这种 
      //$results = Seller::paginate($limit,['*'],'page',5); //paginate 控制page
        return response()->json(['code' => 0, 'data' => $results->items(), 'count' => $results->total()]);
    }
```
