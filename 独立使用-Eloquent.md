---
title: 独立使用 Eloquent
date: 2018-07-04 14:14:22
tags: [Eloquent,教程,Laravel,PHP]
categories: [Laravel]
keywords: [Eloquent,教程,Laravel,PHP]
---
## 前言
Laravel 的 Eloquent 是一个很棒的ORM，Eloquent 是独立的模块，我们也可以在自己的项目里通过composer来使用Eloquent，本文就详细讲解如何在自己的项目集成 Eloquent。

## 安装
在项目目录下执行
```php
composer require illuminate/database:^5.0
```
或直接在 `composer.json` 文件里添加:
```php
{
  "require": {
    "illuminate/database": "^5.0",
    "illuminate/pagination": "^5.0"
  },
  "repositories": {
    "packagist": {
      "type": "composer",
      "url": "https://packagist.phpcomposer.com"    //中国镜像站
    }
  }
}
```

这样 Eloquent 就安装好了。

## 配置
在项目入口文件加入
```php
        
        // 载入 composer 的 autoload 文件
        require_once IA_ROOT . '/addons/guangqian_shop/vendor/autoload.php';
        
        $db_config=$_W['config']['db'];
        //数据库配置
        $database = [
            'driver'    => 'mysql',
            'host'      => '127.0.0.1',
            'database'  => 'test',
            'username'  => 'test',
            'password'  => '123456',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => $db_config['tablepre'],
        ];

        $Capsule = new Capsule;
        //创建数据库链接
        $Capsule->addConnection($database);
        // 设置全局静态可访问
        $Capsule->setAsGlobal();
        // 启动Eloquent
        $Capsule->bootEloquent();

```

至此 Eloquent 就配置完成了


## 使用

### DB 使用
在使用处
+ 确保 Eloquent 的初始化文件已被引入
+ `use Illuminate\Database\Capsule\Manager as DB;`  
即可使用
如:
`db.php`
```php
<?php
//在入口处已经引入过初始化文件
use Illuminate\Database\Capsule\Manager as DB;

class DbGqModel extends DB{

}
```

`controller.php`

```php
<?php
use \DbGqModel;

$users = DbGqModel::select('select * from user');

```

### Model 使用

只要你的模型继承 Eloquent 的 Model 类，就好了：
```php
use  Illuminate\Database\Eloquent\Model  as Eloquent; 

class User extends  Eloquent 
{
    protected $table = 'users';
}

```
那么你就可以很方便的像在 Laravel 框架里一样使用 Eloquent 了：
```php
// 查询id为2的
$users = User::find(2);

// 查询全部
$users = User::all();

// 创建数据
$user = new User;
$user->username = 'someone';
$user->email = 'some@overtrue.me';
$user->save();

// ... 更多
```

## 后记
参考了安正超写的[在 Laravel 外独立使用 Eloquent](https://www.golaravel.com/post/zai-laravelwai-du-li-shi-yong-eloquent/)