---
title: rest API 实例教程之 token 生成
tags:
  - rest API
  - API
  - lumen
  - php
  - token
  - bearer token
keywords:
  - rest API
  - API
  - lumen
  - php
  - token
  - bearer token
  - 教程
  - 实例
  - Mysql
categories:
  - Lumen
abbrlink: 7f372516
date: 2017-06-21 16:42:09
---
## 项目需求

&emsp;&emsp;实现登陆功能，获取 token  

&emsp;&emsp;**HttpMethod: Post**  
&emsp;&emsp;**URI: /token**  

&emsp;&emsp;**Request Header**  
&emsp;&emsp;**Content-Type: application/x-www-form-urlencoded**

<!--more-->

&emsp;&emsp;**Request Body**  
&emsp;&emsp;**grant_type=password&username=13012345678&password=password123**

&emsp;&emsp;**Response code:**  

|状态码|描述|
|-|-|
|200|成功|
|401|无权限|
|500|其他错误|

&emsp;&emsp;**Response Body:(仅 200 时需要读)**  

```json
{
    "access_token":"boQtj0SCGz2GFGz[...]",
    "token_type":"bearer",
  "expires_in":1209599,
  "uid":12345,
  ".issued":"Mon, 14 Oct 2013 06:53:32 GMT",
    ".expires":"Mon, 28 Oct 2013 06:53:32 GMT"
}
```

## 项目分析

&emsp;&emsp;前端会提交一个 `POST` 请求，提交地址是 `/token` ，在类型为`application/x-www-form-urlencoded` 的 `Header` 发送。在 `Body` 中含有用户名与密码参数。

&emsp;&emsp;那么我们需要做的就是从 `/token` 的 `POST` 请求中收到这两个参数，对用户进行验证，如果通过验证就生成一个 `token` 就存于数据库中，并设置过期时间，到期自动删除这个 `token` ，如果已经获取过，也返回 `401` 状态码，如果全部正确并没有获取过，返回以上 `JSON` 。

## 项目开始

### 模型及路由设置

&emsp;&emsp;参考上一遍教程[使用 lumen 创建 rest API 实例教程](http://blog.adminchao.com/使用-lumen-创建-rest-API-实例教程.html),在此不再赘述。  

### 数据库构建

&emsp;&emsp;对项目进行分析后，决定创建以下字段：  

|字段名|类型|属性|作用|
|-|-|-|-|
|id|int|主键|便于操作|
|uid|int|not null|user 表用户的 id ，即为用户的 uid|
|uesr_token|varchar|not null|生成的 token 存放地|
|issued_time|timestamp|not null ,CURRENT_TIMESTAMP|插入记录时间,也可理解为上次获得 token 时间|
|expire_time|timestamp|not null|插入记录时间 + 1209599 , token 过期时间|

&emsp;&emsp;以此创建 `<date>_create_table_token.php` ,并编写以下代码：  

```php
<?php

 use Illuminate\Support\Facades\Schema;
 use Illuminate\Database\Schema\Blueprint;
 use Illuminate\Database\Migrations\Migration;

class CreateTableToken extends Migration
{
    /**
      * Run the migrations.
      *
      * @return void
      */
     public function up()
     {
         Schema::create('token', function (Blueprint $table) {
             $table->increments('id');
             $table->integer('uid');
             $table->string('user_token');
             $table->timestamp('issued_time');
             $table->timestamp('expire_time');

         });
     }

     /**
      * Reverse the migrations.
      *
      * @return void
      */
     public function down()
     {
         Schema::dropIfExists('token');
     }
 }
 ?>
```

### 控制器编写

&emsp;&emsp;创建 `TokenController.php` ，代码中我都加了实现过程的注释，注意看哦  
&emsp;&emsp;如下：  

```php
 <?php

  namespace App\Http\Controllers;

  use App\Token;
  use Illuminate\Http\Response;
  use Illuminate\Http\Request;
  use Illuminate\Support\Facades\DB;
  use Illuminate\Support\Facades\Crypt;

 class TokenController extends Controller
 {
     public  $username;
     public  $password;

     public function __construct(Request $request){
       $this->username = $request->input('username');
       $this->password = $request->input('password');
     }
     public function get_token(Request $request)
     {
       //根据提交的查询
       $arr = DB::select('select * from user where phone=:username',['username'=>$this->username]);
       //如果没有注册
       if(!$arr){
         return response('',401)->header('Content-Type','text/html;charset=utf-8');
       }
       //注册过，对密码进行解密
       $depass = Crypt::decrypt($arr[0]->password);
       //如果提交的密码与解密后的密码一致
       if($this->password === $depass){
         //查询token表，判断是否获取过token
         $ishave = DB::select('select * from token where uid=:uid',['uid'=>$arr[0]->id]);
         //如果获取过
         if($ishave){
           return response('',401)->header('Content-Type','text/html;charset=utf-8');
         }
         //否则
         else{
           //生成token
           $token = md5(md5(microtime(true)));
           //插入uid,和token，其他两个字段在后边mysql有插入方法
           DB::insert('insert into token (uid,user_token) values (:uid,:user_token)',['uid'=>$arr[0]->id,                      'user_token'=>$token]);
           //准备返回的数据
           $return_json = DB::select('select * from token where uid=:uid',['uid'=>$arr[0]->id]);
           //时间差8小时，还原
           $issued_time = strtotime($return_json[0]->issued_time)+28800;
           $expire_time = strtotime($return_json[0]->expire_time)+28800;
           //返回json
           return response()->json(['access_token'=>$return_json[0]->user_token,
                                    'token_type'=>'bearer',
                                    'expires_in'=>'1209599',
                                    'uid'=>$return_json[0]->uid,
                                    '.issued'=>gmdate(DATE_RFC822,$issued_time),
                                    '.expires'=>gmdate(DATE_RFC822,$expire_time)],200);

         }
         //如果密码不一致
       }else{
         return response('',401)->header('Content-Type','text/html;charset=utf-8');
       }
     }
 }
 ?>
```

#### 遇到的问题

&emsp;&emsp;在控制器编写过程中有几个问题要注意：

##### 怎么访问 Mysql 语句执行的结果

&emsp;&emsp;怎么访问 `Mysql` 语句的执行结果呢？官方文档没有详细说明，只是用了一个 `foreach` 来遍历。自己 `var_dump` 了一下，发现执行完成后结果为数组下包含对象，相当于二维数组，但是第二维是个对象，由此推断访问具体值应该是 `$array[0]->property` ，果然如此。

##### 直接使用数据库中的加密密码查询不到数据

&emsp;&emsp;项目过程中刚开始我直接使用用户提交的 `password` 加密过后进行数据库查询，匹配不到结果，后来测试发现 `lumen` 的加密函数 `Crypt::encrypt()` 可能封装有时间戳，每次加密过后的值都不一样，所以想到对数据的密码进行 `Crypt::decrypt()` 解密后，与用户输入的值进行对比。

##### 数据库时间与 PHP 返回的时间不一致

&emsp;&emsp;测试过程中发现数据库时间是正确的，但是返回到前端的 `json` 时间比数据库中的时间提前了 8 个小时由此想到是时区设置问题，用 `php` 自带日前字符串转为时间戳函数 `strtotime($str)` 来解决。

### Mysql 相关

#### 触发器的编写

&emsp;&emsp;在插入记录时，由于 `id` 、 `issued_time` 字段是自动插入的，并且 `expire_time` 字段与 `issued_time` 字段有必然联系，即：在 `issued_time` 的基础上增加 `1209599` 秒，所以想到了建立一个触发器，在 `insert` 之后更新 `expire_time` 字段为 `issued_time` 加 `1209599` 秒，在 `Mysql` 上测试不成功，原因可能是对刚刚插入的记录进行了 `update` ，造成了循环调用，那就用 `set` 来操作，但是 `set` 必须在 `insert` 之前来执行，换成 `set new.expire_time = date_add(new.issued_time,interval 1209599 second)` 后，还是提示错误，原因是 `new.issued_time` 没有不能参与运算，因为还没有成功插入这个值呢，那就使用 `Mysql` 的 `now()` 函数，最终代码如下：  

```sql
create trigger expiretime before insert
          on token for each row
          set new.expire_time = date_add(now(),interval 1209599 second);
```

&emsp;&emsp;代码解释如下：

**创建 触发器 `expiretime` 在插入之前  
在 `token` 表上跟踪记录  
设置 新增的 `expire_time` 值为 现在的时间增加 `1209599` 秒。**  

&emsp;&emsp;`date_add` 函数是向日期添加指定的时间间隔。  

&emsp;&emsp;测试只插入 `uid` 与 `user_token` 字段后，其他字段成功自动生成。  

#### 定时器的编写

&emsp;&emsp;牵扯到 `token` 的有效期，所以要定时删除过期的记录，想到用定时器来实现。  
&emsp;&emsp;首先编写一个执行方法  
&emsp;&emsp;编写代码如下:  

```sql
mysql>  delimiter $
        create procedure `delete_expires`()
        begin

          delete from token where expire_time < now();

        end; $
        delimiter ;
```

&emsp;&emsp;开启 `Mysql` 的 `event` :  
&emsp;&emsp;在 `/etc/mysql/my.cnf` 的 `[mysqld]` 中添加如下代码：

```
event_scheduler = 1
```

&emsp;&emsp;重启 `Mysql` 服务器。查看是否开启了 `event` 。  

```sql
mysql> show variables like 'event_scheduler';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| event_scheduler | ON    |
+-----------------+-------+
1 row in set (0.00 sec)
```

&emsp;&emsp;编写 `event` 计划，如下：

```sql
mysql>    create event `delexp`
                 on schedule every 30 second
                 on completion preserve
                 do
                 call delete_expires();
```

&emsp;&emsp;代码解释：

&emsp;&emsp;`on schedule` 调度时间：可以定义事件被触发的具体时间，也可以指定事件被触发的周期，如果指定周期性任务，还可以指定开始时间和结束时间。  
&emsp;&emsp;`on completion` 事件完成后的行为：默认为 `not preserve` ，即事件完成以后删除任务本身，注意：对于周期性的任务，不表示事件只做一次，而是指到达结束时间后，再删除这个事件。可以将它指定为 `preserve` 那么当事件完成以后，不会被删除。  
&emsp;&emsp;`do call` 事件体，就是事件具体需要做什么，定义方式和存储过程类似。  

&emsp;&emsp;查看运行的 `event` 进程：  

```sql
mysql> show processlist\G
```

#### 删除

&emsp;&emsp;类似于删除表：  

```sql
mysql> drop procedure delete_expires;
Query OK, 0 rows affected (0.00 sec)

mysql> drop event delexp;
Query OK, 0 rows affected (0.01 sec)
```

## 测试

&emsp;&emsp;使用 `Postman` 测试，成功！
