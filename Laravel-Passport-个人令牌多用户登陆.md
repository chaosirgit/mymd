---
title: Laravel Passport 个人令牌多用户登陆
tags:
  - Laravel
  - Passport
  - Oauth
  - php
categories:
  - Laravel
keywords:
  - Passport
  - Laravel
  - Oauth
abbrlink: a681b356
date: 2020-03-27 21:08:27
---

## 前言

最近有个项目需要用到多用户表系统认证，对于 Token 的发放和鉴权，使用了 Passport 来实现 API 授权认证，但是 Passport 对于多用户表登陆实现还是比较难的，在网上到的一些多用户表登陆也都是用 GuzzleHttp 携带额外参数来实现的，不太满足我的需求。经过了一段时期的摸索，终于实现了 Passport 通过个人令牌来多用户登陆。

## 实现

Laravel 版本 6.0

### 安装

```php
composer require laravel/passport
```

### 导出默认迁移文件

```php
php artisan vendor:publish --tag=passport-migrations
```

运行该命令会在 `\app\database\migrations\` 生成

* Date_create_oauth_auth_codes_table.php
* Date_create_oauth_access_tokens_table.php
* Date_create_oauth_refresh_tokens_table.php
* Date_create_oauth_clients_table.php
* Date_create_oauth_personal_access_clients_table.php

五个数据库迁移文件，其中 `Date_create_oauth_access_tokens_table` 是用来记录发放成功的 Token 的。我们需要拷贝一个这个表用来建立另一个用户表的 Token 记录。

### 建立自定义 access_token 表

```
php artisan make:migration create_oauth_other_tokens --create=oauth_other_tokens
```

生成 `Date_create_oauth_other_tokens` 迁移文件。

复制 `Date_create_oauth_access_tokens_table` 文件内容到 `Date_create_oauth_other_tokens`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateOauthOtherTokens extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('oauth_other_tokens', function (Blueprint $table) {
            $table->string('id', 100)->primary();
            $table->unsignedBigInteger('user_id')->nullable()->index();
            $table->unsignedBigInteger('client_id');
            $table->string('name')->nullable();
            $table->text('scopes')->nullable();
            $table->boolean('revoked');
            $table->timestamps();
            $table->dateTime('expires_at')->nullable();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('oauth_other_tokens');
    }
}

```

### 运行数据库迁移

```php
php artisan migrate
```

### 生成客户端

生成两个 `personal` 客户端，一个用与 User 用户，一个用于 Other 用户

```shell
#1
php artisan passport:client --personal

What should we name the personal access client? []:
 > User

Personal access client created successfully.
Client ID: 1
Client secret: 7KqVA8gPxRhPtFdfdsfsuVi4n3xpUBOEiNW4lPI

#2
php artisan passport:client --personal

What should we name the personal access client? []:
 > Other

Personal access client created successfully.
Client ID: 2
Client secret: 7KqVA8gPxRhPtFdfdsfsuVi4n3xpUBOEiNW4fde

```

### 创建自定义 access_tokens 模型

```php
php artisan make:model OtherToken
```

`\app\OtherToken.php`

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Laravel\Passport\Token;

//这里要注意继承 Laravel\Passport\Token
class MiniToken extends Token
{
    protected $table = 'oauth_other_tokens';


    public function user()
    {
        $provider = config('auth.guards.other.provider');

        return $this->belongsTo(config('auth.providers.'.$provider.'.model'));
    }
}

```



### 配置

`\config\auth.php` 配置另一个用户模型

```php
   'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
//            'hash' => false,
        ],
        'other' => [
            'driver' => 'passport',
            'provider' => 'others',
        ],
    ],

'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\User::class,
        ],
        'others' => [
            'driver' => 'eloquent',
            'model' => App\Other::class,
        ],

        // 'users' => [
        //     'driver' => 'database',
        //     'table' => 'users',
        // ],
    ],
```

### 创建中间件

对两个用户路由分别创建中间件，此中间件主要用来设置 Passport 相对应的 Token 模型

```php
php artisan make:middleware UserPassport
php artisan make:middleware OtherPassport
```

在中间件设置响应 Token 模型

`\app\Http\Middleware\UserPassport.php` 

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Laravel\Passport\Passport;

class UserPassport
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
      	//这里用原来的 Token 模型
        Passport::useTokenModel('Laravel\Passport\Token');
      	//设置 ClientId
        Passport::personalAccessClientId(config('auth.clients.api'));
        return $next($request);
    }
}

```

`\app\Http\Middleware\OtherPassport`

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Laravel\Passport\Passport;

class MiniPassport
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
      	//使用自定义 Token 模型
        Passport::useTokenModel('App\MiniToken');
      	//设置 ClientId
        Passport::personalAccessClientId(config('auth.clients.other'));
        return $next($request);
    }
}
```

### 注册中间件

`app\Http\Kernel.php`

```php
protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:60,1',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
						\App\Http\Middleware\ApiPassport::class,
        ],

        'other' => [
            'throttle:60,1',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
            \App\Http\Middleware\MiniPassport::class,
        ],
    ];
```

### 设置 Token 过期时间

`app\Providers\AuthServiceProvider.php`

```php
<?php

namespace App\Providers;

use Carbon\Carbon;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;
use Laravel\Passport\Client;
use Laravel\Passport\Passport;
use Laravel\Passport\RouteRegistrar;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        // 'App\Model' => 'App\Policies\ModelPolicy',
    ];

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();
        //默认令牌发放的有效期是永久

        Passport::personalAccessTokensExpireIn(Carbon::now()->addWeeks(2));
    }
}

```

### 模型 Use Trait

`User` 模型 和 `Other` 模型都要 `Use HasApiTokens`

```php
class Other extends Authenticatable
{
    use HasApiTokens,Notifiable;

    protected $table = 'drivers';
    protected $appends = ['user_name','company_name'];


    public function getUserNameAttribute(){
        return $this->hasOne('App\User','id','user_id')->value('email');
    }

    public function getCompanyNameAttribute(){
        return $this->hasOne('App\company','id','company_id')->value('company_code');
    }

}

// User 模型类似
```

### 格式化返回信息

验证失败返回格式化 json 数据

在 `\app\Exceptions\Handler.php` 中

```php
public function render($request, Exception $exception)
    {
  			//判断路由
        if ($request->is('api/*') || $request->is('other/*')){
          	//判断如果是提交数据验证错误
            if ($exception instanceof ValidationException){
              //$this->error 是自己封装的一个 Trait 返回 json 数据，您也可以自己封装，这里不再展示
                return $this->error(current($exception->errors())[0],42200,$exception->status);
            //判断如果是鉴权错误
            }elseif($exception instanceof AuthenticationException){
               //$this->error 是自己封装的一个 Trait 返回 json 数据，您也可以自己封装，这里不再展示
                return $this->error('授权失败',40100,401);
            }
        }
        return parent::render($request, $exception);
    }
```





## 使用

自定义路由文件自己解决哦 ^_^

### 路由

`\routes\api.php`

```php
<?php

use Illuminate\Http\Request;

/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
|
| Here is where you can register API routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| is assigned the "api" middleware group. Enjoy building your API!
|
*/
Route::prefix('v1')->group(function (){
    //无需授权的api
    Route::get('login', 'UserController@login');

    //需要授权的api
    Route::middleware('auth:api')->group(function (){
        Route::get('/user',function(Request $request){
            return auth()->user();
        });
    });
});



```

`\routes\other.php`

```php
<?php

use Illuminate\Http\Request;

/*
|--------------------------------------------------------------------------
| Other Routes
|--------------------------------------------------------------------------
|
| Here is where you can register Other routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| is assigned the "other" middleware group. Enjoy building your Other!
|
*/
Route::prefix('v1')->group(function (){
    Route::get('/login','OtherController@login');
    //需要授权的 other
    Route::middleware('auth:other')->group(function (){
        Route::get('/other',function(Request $request){
            return auth()->user();
        });
    });

});



```





### User 用户发放 Token

```php
<?php

namespace App\Http\Controllers;
use App\User;


class UserController extends Controller
{



    public function login(LoginPost $request)
    {
      $user = User::find(1);
       return $user->createToken('Api',['*'])->accessToken;
    }

}

```

### Other 用户发放 Token

```php
<?php

namespace App\Http\Controllers;
use App\Other;


class UserController extends Controller
{



    public function login(LoginPost $request)
    {
      $other = Other::find(1);
       return $other->createToken('Other',['*'])->accessToken;
    }

}
```

### 根据 Token 获取 User 用户信息

路由中已写明 请求时注意 Headers 携带 Authorization = Bearer $token



## 测试

### Get api/v1/login

```javascript
{
    "success": true,
    "error_code": 0,
    "message": "请求成功",
    "data": {
        "token_type": "Bearer",
        "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiIxIiwianRpIjoiZGQ2ZmMzNjYwNmU4YjNhNWU1YWQxODMyNDRlNWNlM2JiY2FkNTFkNjc2MGJkZjUxOWMwNzFkNzdiZDY5YjAzZmNlNjc1ZTM0NmQ1MWNkNDUiLCJpYXQiOjE1ODUzMzQwNzgsIm5iZiI6MTU4NTMzNDA3OCwiZXhwIjoxNTg2NTQzNjc3LCJzdWIiOiIyIiwic2NvcGVzIjpbIioiXX0.sQKSS9WCdhMp8_5GTSkQ_sqGMiTVxdVXomub3i3DI3h1xCoAPWYH_rj8W8uTjlX82wzIsFjn0bSKhqTeZFRQDFZWYN-2MatgBk-i6P0dxm-x97sIPfCRMm-omkXIvdWjeJyt..."
    }
}
```

### Get other/v1/login

```JavaScript
{
    "success": true,
    "error_code": 0,
    "message": "请求成功",
    "data": {
        "token_type": "Bearer",
        "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiIyIiwianRpIjoiZmM0NDQzZmQwZmEwYmMzMTNjM2JlMmE5NjUyNzBiYjc5Y2IwMDBmODM0YzkwNzg1M2U1Y2E1NWY5ZDJjODNhOTM5Y2M0YTU1Mjg4MTJlNTQiLCJpYXQiOjE1ODUzMzQxNDUsIm5iZiI6MTU4NTMzNDE0NSwiZXhwIjoxNTg2NTQzNzQ1LCJzdWIiOiIxIiwic2NvcGVzIjpbXX0.LCXpbAQnFTatXyDJtMZmFitt6mo4_h42CqOd3hbnx..."
    }
}
```

### Get api/v1/user

* 携带 `api/v1/login` 返回的 Token

```javascript
{
    "id": 2,
    "email": "test@163.com",
    "sms_verified_at": null,
    "created_at": "2020-03-15 23:58:39",
    "updated_at": "2020-03-15 23:58:39",
    "check_company_status": 2
}
```

* 携带 `other/v1/login` 返回的 Token

```javascript
{
    "success": false,
    "error_code": 40100,
    "message": "授权失败",
    "data": []
}
```



### Get other/v1/other

* 携带 `api/v1/login` 返回的 Token

```javascript
{
    "id": 1,
    "user_id": 1,
    "name": "测试",
    "phone": "138****0869",
    "created_at": "2020-03-20 00:32:55",
    "updated_at": "2020-03-20 00:32:55",
    "user_name": "adminchaochao@163.com",
    "company_name": "河南模因网络科技有限公司"
}
```

* 携带 `other/v1/login` 返回的 Token

```javascript
{
    "success": false,
    "error_code": 40100,
    "message": "授权失败",
    "data": []
}
```

### 问题

由此可见并不能满足需求，使用 Api 发放的 Token 可以获得到两个用户的信息，而 Other 发放的 Token 全是授权失败，Bye Bye~!!

哈哈，开玩笑了啦，在这里要万分感谢我的一个大神朋友～他帮忙找了一下午的源码，最后调试了中间件没执行，原来中间件执行是有顺序的，那么更改一下中间件执行顺序即可：

`\app\Http\Kernel.php`

```php
		//找到这个...
    protected $middlewarePriority = [
        //把刚才定义的两个设置 Token 模型的中间件提前
      	// 1
        \App\Http\Middleware\OtherPassport::class,
        // 2
        \App\Http\Middleware\ApiPassport::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\Authenticate::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];
```

## 完成

### Get api/v1/user

* 携带  `api/v1/login`  返回的 Token

```javascript
{
    "id": 2,
    "email": "test@163.com",
    "sms_verified_at": null,
    "created_at": "2020-03-15 23:58:39",
    "updated_at": "2020-03-15 23:58:39",
    "check_company_status": 2
}
```

* 携带 `other/v1/login` 返回的 Token

```javascript
{
    "success": false,
    "error_code": 40100,
    "message": "授权失败",
    "data": []
}
```

### Get other/v1/other

* 携带 `api/v1/login` 返回的 Token

```javascript
{
    "success": false,
    "error_code": 40100,
    "message": "授权失败",
    "data": []
}
```

* 携带 `other/v1/login` 返回的 Token

```javascript
{
    "id": 1,
    "user_id": 1,
    "name": "测试",
    "phone": "138****0869",
    "created_at": "2020-03-20 00:32:55",
    "updated_at": "2020-03-20 00:32:55",
    "user_name": "adminchaochao@163.com",
    "company_name": "河南模因网络科技有限公司"
}
```

### 以上 Done

谢谢，如果有什么更好的方法请留言哦～