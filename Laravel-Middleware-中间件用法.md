---
title: Laravel Middleware 中间件用法
tags:
  - Laravel
  - Middleware
  - 中间件
  - 实例
categories:
  - Laravel
abbrlink: 4f74b53d
date: 2018-06-08 11:26:34
---

## 前言
Laravel 中间件非常好用。

## 用法

### 注册中间件
打开 `app/Http/Kernel.php` 解释如下:
```php
class Kernel extends HttpKernel
{
    /**
     * 全局中间件
     */
    protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        //\App\Http\Middleware\VerifyCsrfToken::class,
        \Lib\ClusterSession\Middleware\StartSession::class,
    ];

    /**
     * 路由中间件
     */
    protected $routeMiddleware = [
        'access_control' => \App\Http\Middleware\AccessControl::class,
        'api' => \App\Http\Middleware\Api::class,
        'auth' => \App\Http\Middleware\Authenticate::class, //别名 (用这个举例)
        'admin_auth' => \App\Http\Middleware\AdminAuthenticate::class,
        'server_auth' => \App\Http\Middleware\ServerAuthenticate::class,
        'CORS' => \App\Http\Middleware\CORS::class
    ];
}
```

### 创建中间件
 创建 `app/Http/Middleware/Authenticate.php` 文件，如下：
 ```php
 class Authenticate
 {
     public function handle($request, Closure $next)
     {
         $value = SessionManager::get('user_id', null);
 
         if(empty($value)){
             return response()->json(['error' => '999', 'message' => '请先登录']);
         }
 
         return $next($request); //如果通过，去下一个请求
     }
 
 
 }
 ```
 
 ### 使用中间件
 在 `routes.php` 中使用，如下：
 ```php
 
 // prefix 路由分组,middleware 中间件
 Route::group(['prefix' => 'api', 'middleware' => ['access_control', 'api', 'CORS']], function () {
     Route::post('/user/info', ['uses' => 'Api\UserController@info', 'middleware' => ['auth']]);//用户中心
     Route::get('/user/address', ['uses' => 'Api\UserController@UserAddress', 'middleware' => ['auth']]);//个人收货地址列表
     Route::get('/user/address/info', ['uses' => 'Api\UserController@UserAddressInfo', 'middleware' => ['auth']]);//获取收货地址详情
     Route::post('/user/address/post', ['uses' => 'Api\UserController@UserAddressPost', 'middleware' => ['auth']]);//获取收货地址详情
     Route::post('/user/address/delete', ['uses' => 'Api\UserController@UserAddressDelete', 'middleware' => ['auth']]);//获取收货地址详情
     Route::post('/user/modify', ['uses' => 'Api\UserController@modifyUserInfo', 'middleware' => ['auth']]);//获取收货地址详情
         Route::post('/message/send', 'Api\SmsController@send');//发送短信
         Route::post('/upload/configure', 'Api\DefaultController@upload');//发送短信
     
         Route::post('/user/register', 'Api\UserController@register');//用户注册接口
         Route::post('/user/login', 'Api\UserController@login');//用户注册接口
         Route::post('/user/signout', 'Api\UserController@signout');//退出接口
 }
```




