---
title: 使用 Lumen 创建 RestApi 实例教程
tags:
  - RestApi
  - Lumen
  - PHP框架
  - 实例教程
categories:
  - Lumen
abbrlink: 9794a7df
date: 2017-06-09 11:43:31
keywords: [RestApi,Lumen,PHP框架,实例,教程]
---

## 项目需求
开发一个用户注册的 rest API 要求：

HttpMethod: Post  
URI: api/account/register  
**Request Body**

|字段|必选|类型及范围|说明|
|-|-|-|-|
|phone|是|string|注册手机号|
|password|是|string|密码|
|code|是|string|验证码|
|gender|是|enum|性别|
|nickname|否|string|昵称，不超过 20 字符|
|birthday|否|date|生日|
|height|否|int|身高|
|weight|否|int|体重|
|email|否|string|邮箱|

**Response code:**

|状态码|内容|
|-|-|
|200|成功|
|400|失败|
|500|其他未处理错误|

## 项目开始

### 配置数据库
在 lumen 项目根目录下找到 .env.example 拷贝为 .env
```
cp ./.env.example ./.env
```

打开 .env 文件 , 配置数据库连接信息(数据库名等不加引号)
```
DB_DATABASE=<db_name>
DB_USERNAME=<db_username>
DB_PASSWORD=<db_password>
```
然后在 bootstrap/app.php 中取消下面两行之前的注释：

```
$app->withFacades();
$app->withEloquent();
```
### 数据库迁移
接下来我们来创建数据表。  
项目根目录下运行如下命令：
```
php artisan make:migration create_table_user --create=user
```
该命令将会在 `database/migrations` 目录下创建一个迁移文件 `<date>_create_table_user.php`  
接下来我们来编辑这个文件来定义数据表。

```php
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateTableUser extends Migration
{
     /**
      * Run the migrations.
      *
      * @return void
      */
     public function up()
     {
       //以下为自定义修改，其他代码自动生成
         Schema::create('user', function (Blueprint $table) {
             $table->increments('id');
             $table->timestamps();//该方法会自动在表里建立 created_at 与 updated_at;
             $table->string('phone');
             $table->string('password');
             $table->enum('gender',['male','female']);
             $table->string('nickname')->nullable();
             $table->date('birthday')->nullable();//使用列修改器设置允许为null;
             $table->integer('height')->nullable();
             $table->integer('weight')->nullable();
             $table->string('email')->nullable();
         });
      //以上为自定义修改，其他代码自动生成
     }
     /**
      * Reverse the migrations.
      *
      * @return void
      */
     public function down()
     {
         Schema::dropIfExists('user');
     }
 }
 ?>
```

#### 列修改器

把必选为否的字段用列修改器设置允许为 `null` ，列修改器列表：

|修改器|描述|
|-|-|
|->first()|将该列置为表中第一个列 (仅适用于MySQL)|
|->after(‘column’)|将该列置于另一个列之后 (仅适用于MySQL)|
|->nullable()|允许该列的值为NULL|
|->default($value)|指定列的默认值|
|->unsigned()|设置 integer 列为 UNSIGNED|

现在我们来运行这个迁移：

```
php artisan migrate
```

这样，就会在数据库中创建对应的表。  
查询这个表的字段看看:
```
mysql> show columns from user;
+------------+-----------------------+------+-----+---------+----------------+
| Field      | Type                  | Null | Key | Default | Extra          |
+------------+-----------------------+------+-----+---------+----------------+
| id         | int(10) unsigned      | NO   | PRI | NULL    | auto_increment |
| created_at | timestamp             | YES  |     | NULL    |                |
| updated_at | timestamp             | YES  |     | NULL    |                |
| phone      | varchar(255)          | NO   |     | NULL    |                |
| password   | varchar(255)          | NO   |     | NULL    |                |
| gender     | enum('male','female') | NO   |     | NULL    |                |
| nickname   | varchar(255)          | YES  |     | NULL    |                |
| birthday   | date                  | YES  |     | NULL    |                |
| height     | int(11)               | YES  |     | NULL    |                |
| weight     | int(11)               | YES  |     | NULL    |                |
| email      | varchar(255)          | YES  |     | NULL    |                |
+------------+-----------------------+------+-----+---------+----------------+
11 rows in set (0.00 sec)
```

#### 常用的字段类型

|命令|描述|
|-|-|
|$table->bigIncrements(‘id’)|递增 ID（主键），相当于「UNSIGNED BIG INTEGER」型态。|
|$table->boolean(‘confirmed’)|相当于 BOOLEAN 型态。|
|$table->char(‘name’, 4)|相当于 CHAR 型态，并带有长度。|
|$table->date(‘created_at’)|相当于 DATE 型态。|
|$table->dateTime(‘created_at’)|相当于 DATETIME 型态。|
|$table->dateTimeTz(‘created_at’)|DATETIME (带时区) 形态
|$table->decimal(‘amount’, 5, 2)|相当于 DECIMAL 型态，并带有精度与基数。|
|$table->double(‘column’, 15, 8)|相当于 DOUBLE 型态，总共有 15 位数，在小数点后面有 8 位数。|
|$table->enum(‘choices’, [‘foo’, ‘bar’])|相当于 ENUM 型态。|
|$table->float(‘amount’, 8, 2)|相当于 FLOAT 型态，总共有 8 位数，在小数点后面有 2 位数。|
|$table->increments(‘id’)|递增的 ID (主键)，使用相当于「UNSIGNED INTEGER」的型态。|
|$table->integer(‘votes’)|相当于 INTEGER 型态。|
|$table->ipAddress(‘visitor’)|相当于 IP 地址形态。|
|$table->json(‘options’)|相当于 JSON 型态。|
|$table->jsonb(‘options’)|相当于 JSONB 型态。|
|$table->longText(‘description’)|相当于 LONGTEXT 型态。|
|$table->mediumIncrements(‘id’)|递增 ID (主键) ，相当于「UNSIGNED MEDIUM INTEGER」型态。|
|$table->mediumInteger(‘numbers’)|相当于 MEDIUMINT 型态。|
|$table->mediumText(‘description’)|相当于 MEDIUMTEXT 型态。|
|$table->nullableTimestamps()|与 timestamps() 相同，但允许为 NULL。|
|$table->rememberToken()|加入 remember_token 并使用 VARCHAR(100) NULL。|
|$table->smallIncrements(‘id’)|递增 ID (主键) ，相当于「UNSIGNED SMALL INTEGER」型态。|
|$table->smallInteger(‘votes’)|相当于 SMALLINT 型态。|
|$table->softDeletes()|加入 deleted_at 字段用于软删除操作。|
|$table->string(‘email’)|相当于 VARCHAR 型态。|
|$table->string(‘name’, 100)|相当于 VARCHAR 型态，并带有长度。|
|$table->text(‘description’)|相当于 TEXT 型态。|
|$table->time(‘sunrise’)|相当于 TIME 型态。|
|$table->timeTz(‘sunrise’)|相当于 TIME (带时区) 形态。|
|$table->tinyInteger(‘numbers’)|相当于 TINYINT 型态。|
|$table->timestamp(‘added_on’)|相当于 TIMESTAMP 型态。|
|$table->timestamps()|加入 created_at 和 updated_at 字段。|
|$table->unsignedInteger(‘votes’)|相当于 Unsigned INT 型态。|
|$table->unsignedMediumInteger(‘votes’)|相当于 Unsigned MEDIUMINT 型态。|
|$table->unsignedSmallInteger(‘votes’)|相当于 Unsigned SMALLINT 型态。|
|$table->unsignedTinyInteger(‘votes’)|相当于 Unsigned TINYINT 型态。|
|$table->uuid(‘id’)|相当于 UUID 型态。|

### 创建模型

接下来我们在 `app` 目录下创建模型文件 `User.php`，并编写代码如下：
```php
<?php
 namespace App;
 use Illuminate\Database\Eloquent\Model;
 class User extends Model
 {
         protected $fillable =
          ['phone','password','gender','nickname','birthday','height','weight','email'];
 }
 ?>
```

### 创建控制器

然后创建控制器文件 `app/Http/Controllers/UserController.php` :
```php
<?php
namespace App\Http\Controllers;
use App\User;
use Illuminate\Http\Request;
use Illuminate\Http\Response; //这个会自动引用
use Illuminate\Support\Facades\Crypt;//加密所需门面
use Illuminate\Support\Facades\DB;//引入数据库操作门面
class UserController extends Controller
{
         public function register (Request $request){
         $phone = $request->input('phone');
         $password = $request->input('password');
         $code= $request->input('code');
         $gender= $request->input('gender');
         $nickname= $request->input('nickname') ?? null;
         $birthday= $request->input('birthday') ?? null;
         $height= $request->input('height') ?? null;
         $weight= $request->input('weight') ?? null;
         $email= $request->input('email') ?? null;
         //如果验证码正确
         if($code == '888888'){
                 if(!preg_match('/^\d{11}$/',$phone)){
                 //return 'phone flase <br>';}
                 return response('请填写正确的手机号',400)->header('Content-Type','text/html;charset=utf-8');}
                 if(!preg_match('/^[0-9a-zA-Z\S]{6,16}$/',$password)){
                 //return 'password is 6-16 <br>';}
                 return response('密码必须是6-16位字母or数字or符号',400)->header('Content-Type','text/html;charset=utf-8');}
                 if($gender == null){
                 //return 'select gender <br>';}
                 return response('性别不能为空',400)->header('Content-Type','text/html;charset=utf-8');}
                 if($nickname != null && !preg_match('/^[\x{4e00}-\x{9fa5}A-Za-z0-9]{1,20}$/u',$nickname)){
         //      return 'nickname mast <=20 <br>';}
                 return response('昵称不能超过20个字符且不能含有符号',400)->header('Content-Type','text/html;charset=utf-8');}
                 if($birthday != null && !preg_match('/\d{4}-\d{2}-\d{2}/',$birthday)){
                 //return 'birthday: yyyy-mm-dd <br>';}
                 return response('请填写正确的生日格式',400)->header('Content-Type','text/html;charset=utf-8');}
                 if($height != null && !preg_match('/^\d{2,3}$/',$height)){
                 //return 'height is 2-3bit number <br>';}
                 return response('请填写正确的身高',400)->header('Content-Type','text/html;charset=utf-8');}
                 if($weight != null && !preg_match('/^\d{2,3}$/',$weight)){
                 //return 'weight is 2-3bit number <br>';}
                 return response('请填写正确的体重',400)->header('Content-Type','text/html;charset=utf-8');}
                 if($email != null && !preg_match('/\w[-\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\.)+[A-Za-z]{2,14}/',$email)){
                 return response('请填写正确的电子邮箱',400)->header('Content-Type','text/html;charset=utf-8');}
                 //是否存在重复的
                 $userlist = DB::select('select id,phone,email from user where phone = :phone or email = :email',['phone' => $phone,'email' => $email]);
                 if($userlist){
                         return response('您的手机号或邮箱已注册',400)->header('Content-Type','text/html;charset=utf-8');
                 }else{
                 DB::insert('insert into user (created_at,phone,password,gender,nickname,birthday,height,weight,email) values (:created_at,:phone,:            password,:gender,:nickname,:birthday,:height,:weight,:email)',['created_at'=>date('Y-m-d H:i:s'),'phone'=>$phone,'password'=>Crypt::encrypt($password),       'gender'=>$gender,'nickname'=>$nickname,'birthday'=>$birthday,'height'=>$height,'weight'=>$weight,'email'=>$email]);
                 return response('注册成功',200)->header('Content-Type','text/html;charset=utf-8');
         //      return 'oK';
                 }
         }else{
         //      return '验证码错误';
                 return response('验证码错误',400)->header('Content-Type','text/html;charset=utf-8');
         }
         }
}
?>
```

#### 遇到的问题

response 的使用方法  

单个:
```php
return response('失败',400)->header('Content-Type','text/html;charset=utf-8');
```

多个:
```php
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');

```

json:  
`json` 方法会自动将标头的 `Content-Type` 设置为 `application/json`，
并通过 `PHP` 的 `json_encode` 函数将指定的数组转换为 `JSON`:
```php
return response()->json(['name' => 'chaosir', 'age' => '30']);
```
DB 操作问题
在使用 `DB` 类来操作数据库时报错，原因是没有引入数据库操作门面：
```php
use Illuminate\Support\Facades\DB;
```
并且打开 `bootstrap\app.php` 取消 `$app->withFacades();` 之前的注释

使用实例:
```php
$userlist = DB::select('select id,phone,email from user where phone = :phone or email = :email',
['phone' => $phone,'email' => $email]);
```
### 定义路由
打开 `/routes/web.php` 设置如下路由：
```php
$app->post('api/account/register','UserController@register');
```
## 项目完成


