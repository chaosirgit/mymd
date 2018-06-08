---
title: Thinkphp5 创建 API 实例教程
tags:
  - tp5
  - ThinkPHP5
  - API
  - 实例
  - 教程
keywords:
  - tp5
  - ThinkPHP5
  - API
  - 实例
  - 教程
categories:
  - ThinkPHP5
abbrlink: 166ddea1
date: 2017-08-04 20:20:03
---
## 前言

现在的开发模式前后端分离已经成了主流，需要用到 API 接口的开发，下面使用 Thinkphp5 开发 API 接口，主要是路由的定制。

<!--more-->

## 项目需求   

*登陆功能*  

```
POST  /admin/v1/login
```

*请求参数说明*  

|字段|类型|描述|是否必要|备注|
|-|-|-|-|-|
|username|string|用户名|是||
|password|string|密码|是|&nbsp;|

*Request Body*  

```
{
  "username":"admin",
  "password":"123456"
}
```

*Response Body*  

```
{
  "code":"0"
  "msg":"登陆失败，请重试!"
  ......
}
```

*返回参数说明*  

|字段|说明|
|-|-|
|code|0为登陆失败，1为登陆成功|
|msg|所匹配的信息|

*注：后端需要设置 Session,以下功能如无特殊注明都是在设置了 Session 的前提下实现。*

## 项目开始

### 建立所需目录

#### 建立 `application\api\controller` 目录。  

#### 建立 `application\api\controller\Admin.php` 文件。  

```
namespace app\api\controller;
use think\Controller;

class Admin extends Controller
{
  public function login(){
    //逻辑自己写，哈哈
  }
}
```

#### Apache 隐藏 `index.php` 路径：  

在 `public` 公共目录下配置 `.htaccess` 文件，改为如下代码：

```
<IfModule mod_rewrite.c>
Options +FollowSymlinks -Multiviews
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ index.php/$1 [QSA,PT,L]
</IfModule>
```

如果是 `phpstudy` ，规则定义如下：

```
<IfModule mod_rewrite.c>
Options +FollowSymlinks -Multiviews
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ index.php [L,E=PATH_INFO:$1]
</IfModule>

```

#### 更改项目访问路径隐藏 `public` 。

#### 配置 `application\route.php` 文件，添加如下代码：  


```php
use think\Route;

Route::rule('admin/v1/login','api/admin/login');
// admin/v1/login 为定义路由的地址
// api/admin/login 为实际地址，api 模块下 admin 控制器 login 方法
```

访问 `localhost/admin/v1/login` 取得返回数据。  
