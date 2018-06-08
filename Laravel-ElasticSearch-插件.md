---
title: Laravel ElasticSearch 插件
tags:
  - Laravel
  - PHP
  - ElasticSearch
categories:
  - Laravel
abbrlink: b39e844c
date: 2018-06-08 12:43:00
---

## ElasticSearch 问题及用法
### 出现 index.max_result_window 报错的解决办法  

```shell
curl -XPUT "http://localhost:9200/my_index/_settings" -d '{ "index" : { "max_result_window" : 100000000 } }'
```

### 搜索封装
```php
public static function SearchAccountLogEs($must = array(),$must_not = array(),$should = array(),$aggs = array(),$size = 10,$page = 1,$sort = array(),$debug = false){
        $index = Config::get('elasticsearch.index');
        $type = 'accountlog';
        $must = $must ?? array();
        $must_not = $must_not ?? array();
        $should = $should ?? array();
        $aggs = $aggs ?? array();
        $sort = $sort ?? array();
        $from = ($page - 1) * $size;
        $params = [
            'index'=>$index,
            'type' => $type,
            'body'=>[
                'query'=>[
                    'bool'=>[
                        'must'=>$must,
                        'must_not'=>$must_not,
//                        'should' => $should
                        ]
                    ]
//                'aggs'=>$aggs,
                ],
            'size' => $size,
            'from' => $from
        ];
        
        $client = ClientBuilder::create()
            ->setHosts( Config::get( 'elasticsearch.hosts' ) )
            ->setRetries( 2 )
            ->build();
        if(!empty($aggs)){
            $params['body']['aggs'] = $aggs;
        }
        if(!empty($should)){
            $params['body']['query']["bool"]["should"] = $should;
            $params['body']['query']["bool"]["minimum_should_match"] = 1;
        }
        if(!empty($sort)){
            $params['body']['sort'] = array($sort);
        }
        if($debug == true)
            return $params;
        $response = $client->search($params);
        $results = array(
          'total' => $response['hits']['total'],
//            key($aggs) => $response['aggregations'][key($aggs)]['value']
        );
        if (!empty($aggs)){
            $results[key($aggs)] = $response['aggregations'][key($aggs)]['value'];
        }
        $results['data'] = [];
        foreach ($response['hits']['hits'] as $key => $value){
            $results['data'][] = $value['_source'];
        }

        return $results;
    }
```

### 更新封装
```php
public static function updateAccountLogEs($log)
    {
        $other = User::find($log->user_id);
        //更新ES索引
        $client = ClientBuilder::create()
            ->setHosts(Config::get('elasticsearch.hosts'))
            ->setRetries(2)
            ->build();
        $index = Config::get('elasticsearch.index');
        $type = 'accountlog';

        $params = [
            'index' => $index,
            'type'  => $type,
            'id'    => $log->id
        ];

        try {
            $client->delete($params);
        }catch (\Exception $ex){}

        $params = [
            'index' => $index,
            'type' => $type,
            'id' => $log->id,
            'body' => [
                'id' => $log->id,
                'user_id' => $log->user_id,
                'user_money' => $log->user_money,
                'user_money1' => $log->user_money1,
                'user_money3' => $log->user_money3,
                'frozen_money' => $log->frozen_money,
                'letter_of_credit' => $log->letter_of_credit,
                'shop_letter_credit' => $log->shop_letter_credit,
                'rank_points' => $log->rank_points,
                'pay_points' => $log->pay_points,
                'created_time' => $log->time,
                'info' => $log->info,
                'province_id' => $other->province_id,
                'city_id' => $other->city_id,
                'county_id' => $other->county_id,
                'industry_id' => $other->industry_id,
                'parent_id' => $other->parent_id,
                'type' => $log->type
            ]
        ];

        $client->index($params);
    }

```
