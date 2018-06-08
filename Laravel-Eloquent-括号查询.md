---
title: Laravel Eloquent 括号查询
tags:
  - Laravel
  - PHP
  - Eloquent
  - 查询
  - 实例
  - 括号
categories:
  - Laravel
abbrlink: dfa2bf5f
date: 2018-06-08 12:36:54
---

相当于 `select * from queue_second_sold where status=0 and (buy_user_id = 341 or (sell_user_id = 341 and buy_user_id != 0)) order by id desc`,用法如下:

```php
    /**
     * 交易大厅-定向交易
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function soldList(Request $request){
        $type = $request->get('type',0);    //交易大厅
        $limit = $request->get('limit',10);
        $user = User::getUserInfo();
        if($type == 1){                     //定向交易
            $results = QueueSecondSold::where('status',0)
            ->where(function($query) use ($user){
                $query->where('sell_user_id',$user->id)
                ->where('buy_user_id','!=',0);
            })->orWhere('buy_user_id',$user->id)
            ->orderBy('id','desc')->paginate($limit);

        }else{
            $results = QueueSecondSold::where('status',0)->where('buy_user_id',0)->orderBy('id','desc')->paginate($limit);
        }

        return $this->success(['result'=>$results->items(),'total'=>$results->total(),'page'=>$results->currentPage(),'pages'=>$results->lastPage(),'user_id'=>$user->id]);
    }

```

