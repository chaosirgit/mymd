---
title: Guzzle 基础用法
tags:
  - Guzzle
  - PHP
categories:
  - Laravel
keywords:
  - Guzzle
  - PHP
  - Laravel
abbrlink: c77b23f0
date: 2019-06-17 23:51:47
---
## 安装
```
composer require guzzlehttp/guzzle:~6.0
```

## 使用
```php
<?php
use GuzzleHttp\Client;


$client = new Client();
$api = 'http://www.test.com/api/register';
$request_data = ['username'=>'111','password'=>'222'];
$sign = md5('key');
$response = $client->request('POST',$api,['form_params'=>$request_data,'headers'=>['Sign'=>$sign]]);
$res_string = $response->getBody()->getContents();
$res_data = json_decode($res_string,true);
var_dump($res_data);

```

## 对请求的封装

```php
<?php
    use GuzzleHttp\Client;
    use GuzzleHttp\Cookie\CookieJar;
    use GuzzleHttp\Cookie\SetCookie;

   public function request($method,$api,$data = array(),$cookie = false){
        $client      = new Client();
        if ($method == 'POST'){
            $response    = $client->request($method, $api, ['form_params' => $data]);
        }elseif($method == 'GET'){
            $response    = $client->request($method, $api);
        }
        $res_string  = $response->getBody()->getContents();

        $res_data    = json_decode($res_string, true);
        if ($cookie){
            $cookie_arr = $response->getHeader('Set-Cookie');
            $res_data['Set-Cookie'] = $cookie_arr[0];
        }
        return $res_data;
    }

    public function cookieRequest($uuid,$method,$api,$cookie_string,$data = array()){
        $cookie_jar = new CookieJar();
        $cookie_jar->setCookie(SetCookie::fromString($cookie_string));
        $client = new Client(['cookies'=>$cookie_jar]);
        if ($method == 'POST'){
            $response    = $client->request($method, $api, ['form_params' => $data]);
        }elseif($method == 'GET'){
            $response    = $client->request($method, $api);
        }
        $res_string  = $response->getBody()->getContents();

        $res_data    = json_decode($res_string, true);

        return $res_data;

    }
    
    ?>
```