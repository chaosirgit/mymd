---
title: Laravel 生成测试数据
tags:
  - Laravel
  - PHP
  - Faker
  - 测试
  - 数据
  - 实例
categories:
  - Laravel
abbrlink: 3346bf17
date: 2018-03-08 11:14:46
keywords: [laravel,PHP,Faker,测试,数据,实例]
---
## 前言
使用 Laravel Faker 生成测试数据，很方便测试调试。

## 用法
### 打开 `app/database/factories/ModelFactory.php`

### 编辑如下：
```php
$factory->define(App\User::class, function (Faker\Generator $faker) {
    $faker = Faker\Factory::create('zh_CN'); //中文包
    return [
        'openid'       => str_random(10),
        'nickname'     => $faker->name,  //中文姓名
        'mobile'       => $faker->phoneNumber,
        'avatar'       => $faker->imageUrl(),     //图片URL地址
        'integral'     => $faker->randomNumber(3), //随机3位整型(0-999)
        'balance'      => $faker->randomFloat(2, 0, 10000), //随机浮点数,2位小数点,最小0，最大10000
        'birthday'     => $faker->date(), //日期
        'created_time' => $faker->unixTime(), //unix时间戳
        'password'     => App\User::generatePassword('haha123'), //可用模型方法生成数据
    ];
});

$factory->define(App\Product::class, function (Faker\Generator $faker) {
    $faker  = Faker\Factory::create('zh_CN');
    //用模型生成要关联随机的数组
    $seller = App\Seller::where('status', 1)->get()->toArray();
    foreach ($seller as $value) {
        $row[] = $value['id'];
    }
    return [
        'product_no'   => $faker->randomNumber(8),
        'name'         => '【测试商品】' . $faker->streetName,
        'cate_id'      => random_int(1, 6), //1-6随机int
        'seller_id'    => $faker->randomElement($row),//随机数组
        'mod_id'       => random_int(1, 10),
        'price'        => $faker->randomFloat(2, 1, 500),
        'weight'       => random_int(1, 100),
        'img'          => $faker->imageUrl('480','480'),//宽480 高480 的图片URL
        'thumb'        => $faker->imageUrl('480','480'),
        'content'      => '【测试商品内容】' . $faker->text(),
        'is_point'     => random_int(0, 1),
        'is_new'       => random_int(0, 1),
        'is_recommend' => random_int(0, 1),
        'is_sale'      => random_int(0, 1),
        'create_time'  => $faker->unixTime(),
    ];
});

$factory->define(App\Seller::class, function (Faker\Generator $faker) {
    $faker = Faker\Factory::create('zh_CN');
    $user  = App\User::all()->toArray();
    foreach ($user as $value) {
        $row[] = $value['id'];
    }
    return [
        'user_id'     => $faker->randomElement($row),
        'seller_name' => $faker->colorName,  //很有意境的中文色彩名
        'corporate'   => $faker->imageUrl(),
        'business'    => $faker->imageUrl(),
        'create_time' => $faker->unixTime(),
    ];
});

```

然后用 php artisan tinker 进入 Laravel 命令行

```php
factory(App\Product::class,50)->create(); //生成 50 条 存入数据库
factory(App\User::class,50)->make();      //生成 50 条 不存入数库
```

