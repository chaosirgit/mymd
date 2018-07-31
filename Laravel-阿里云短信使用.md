---
title: Laravel 阿里云短信使用
tags:
  - Laravel
  - Sms
  - AliYun
  - 短信
keywords:
  - Laravel
  - Sms
  - AliYun
  - 短信
categories:
  - Laravel
abbrlink: fef79e6c
date: 2018-07-31 17:14:34
---

## curder/laravel-aliyun-sms

### 安装
```php
composer require curder/laravel-aliyun-sms
```

### 配置
在 `Laravel` 项目的 `.env` 文件中配置如下信息。  
```php
ALIYUN_SMS_ENABLE_HTTP_PROXY=false
ALIYUN_SMS_HTTP_PROXY_IP=127.0.0.1
ALIYUN_SMS_HTTP_PROXY_PORT=8888
ALIYUN_SMS_REGION_ID=cn-hangzhou
ALIYUN_SMS_AK=""
ALIYUN_SMS_AS=""
ALIYUN_SMS_SIGN_NAME=""
ALIYUN_SMS_VARIABLE=""
ALIYUN_SMS_CODE=""
```
### 注册 ServiceProvide
在项目的 `config/app.php` 文件中 `providers` 数组中新增如下行：  
```php
Curder\LaravelAliyunSms\ServiceProvider::class,
```

### 生成配置文件
```php
php artisan vendor:publish --provider="Curder\LaravelAliyunSms\ServiceProvider"
```

生成的文件在 `config/aliyunsms.php` 可以前往修改

### 用法
```php
<?php
namespace App\Http\Controllers\Api;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Input;
use Illuminate\Support\Facades\Session;


class SmsController extends Controller
{
/**
     * 发送短信
     * @return \Illuminate\Http\JsonResponse
     */
    public function send(Request $request)
    {
        $ALIYUN_SMS_AK = env("ALIYUN_SMS_AK");
        $ALIYUN_SMS_AS = env("ALIYUN_SMS_AS");
        $ALIYUN_SMS_SIGN_NAME = env("ALIYUN_SMS_SIGN_NAME");
        $ALIYUN_SMS_VARIABLE = env("ALIYUN_SMS_VARIABLE");  //内容变量
        $tplId = env('ALIYUN_SMS_CODE');                //模版ID 模版CODE 格式为 SMS_140736882

        if(empty($tplId) || empty($ALIYUN_SMS_AK) || empty($ALIYUN_SMS_AS) || empty($ALIYUN_SMS_SIGN_NAME) || empty($ALIYUN_SMS_VARIABLE))
            return $this->error('系统配置错误，请联系系统管理员');

//        Config::set("aliyunsms.access_key",$ALIYUN_SMS_AK);
//        Config::set("aliyunsms.access_secret",$ALIYUN_SMS_AS);
//        Config::set("aliyunsms.sign_name",$ALIYUN_SMS_SIGN_NAME);
        $mobile = $request->get('mobile','');
        if(empty($mobile))
            return $this->error('手机号不能为空');

        //检查1分钟内该ip是否发送过验证码
//        if ($this->checkSmsIp($request->ip().$mobile)) {
//            return $this->error('验证码发送过于频繁');
//        }

        $verification_code = $this->createSmsCode(6);
        $params = [
            $ALIYUN_SMS_VARIABLE => $verification_code
        ];

        try{
            $smsService = \App::make('Curder\LaravelAliyunSms\AliyunSms');
            $return = $smsService->send(strval($mobile), $tplId , $params);
            //string $mobile -> 短信接受手机号
            //string $tplId -> 模板签名id，需要在阿里云后台申请，并通过审核才可以使用
           // array $params -> 发送参数

            if($return->Message == "OK"){
                //记入session
                session(['sms_captcha'=>$verification_code]);
                session(['sms_mobile'=> $mobile]);

                //设置缓存key
//                $this->setSmsIpKey($request->ip().$mobile, $mobile);
                return $this->success("发送成功");
            }else{
                return $this->error($return->Message);
            }
        }catch (\ErrorException $e){
            return $this->error($e->getMessage());
        }
    }
    
    /**
         * 生成短信验证码
         * @param int $num  验证码位数
         * @return string
         */
        public function createSmsCode($num=6) {
            //验证码字符数组
            $n_array = range(0, 9);
            //随机生成$num位验证码字符
            $code_array = array_rand($n_array, $num);
            //重新排序验证码字符数组
            shuffle( $code_array );
            //生成验证码
            $code = implode('', $code_array);
            return $code;
        }
    
    
    
}
    
    
    
    ?>
```