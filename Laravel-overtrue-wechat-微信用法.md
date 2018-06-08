---
title: Laravel overtrue/wechat 微信用法
tags:
  - Laravel
  - overtrue
  - wechat
  - 微信
  - PHP
  - 实例
categories:
  - Laravel
abbrlink: fbd4f49e
date: 2018-06-08 12:51:35
---

### 网页授权登陆中间件

#### 创建 `app/Http/Middleware/WechatAuth.php` 中间件
```php
    public function handle($request, Closure $next, $account = 'default', $scopes = null)
        {
            // $account 与 $scopes 写反的情况
            if (is_array($scopes) || (\is_string($account) && str_is('snsapi_*', $account))) {
                list($account, $scopes) = [$scopes, $account];
                $account || $account = 'default';
            }
    
            $isNewSession = false;
            $sessionKey = \sprintf('user_id', $account);
            $config = config(\sprintf('wechat.official_account.%s', $account), []);
            $officialAccount = app(\sprintf('wechat.official_account.%s', $account));
            $scopes = $scopes ?: array_get($config, 'oauth.scopes', ['snsapi_base']);
    
            if (is_string($scopes)) {
                $scopes = array_map('trim', explode(',', $scopes));
            }
    
    
            //$session = session([$sessionKey=>'24260']); //测试环境直接赋值session 正式环境注释
            
               /**
                * 1. 是否有 session
                * 2. 没有 session 判断是否有 code
                * 3. 没有 code 获取code-回调到本页
                * 4. 微信返回 code 参数请求本页
                * 5. 有 code 取得 openid 执行数据库操作
                * 6. openid 在数据库里赋值 session 跳到下一步请求
                * 7. openid 不再数据库里 判断能否取到 nickname ,取不到使用 snsapi_userinfo 授权登陆，否则存入相关信息到数据库并赋值 session
                */
            
            if (!session($sessionKey)) {                          //1.没有session
                if ($request->has('code')) {                 //2.有code
    
                    $wechat_user = $officialAccount->oauth->user();         
                    $data['openid'] = $wechat_user->getId() ?? '';          //取得openid
                    $has = UserWechat::getByOpenid($data['openid']);
                    $has_user = User::where('tync_openid',$data['openid'])->first();
                    if(!empty($has) && !empty($has_user)){              //openid 在库里跳转到下一步
                        $user_id = User::where('tync_openid',$data['openid'])->first()->id ?? 0;
                        session([$sessionKey => $user_id ?? '']);
                        return $next($request);
                    }else{                                              //不再库里
                        $data['nickname'] = $wechat_user->getNickname() ?? '';
                        if(empty($data['nickname'])){                   //取不到nickname 使用授权登陆
                            session([$sessionKey => '']);
                            return $officialAccount->oauth->scopes(['snsapi_userinfo'])->redirect($request->fullUrl());
                        }else{                                          //取到nickname 存库
                            $data['headimgurl'] = $wechat_user->getAvatar() ?? '';
                            UserWechat::createUser($data);                  //存入相关信息
                            User::createWeChat($data);
                            $user_id = User::where('tync_openid',$data['openid'])->first()->id ?? 0;
                            session([$sessionKey => $user_id ?? '']);
                        }
                    }
    
                    $isNewSession = true;
    
    //                Event::fire(new WeChatUserAuthorized(session($sessionKey), $isNewSession, $account));
    
                    return redirect()->to($this->getTargetUrl($request));
                }
                /**
                 * 3.没有code 获取code-回调到本页
                 */
    
                session()->forget($sessionKey);
    
                return $officialAccount->oauth->scopes($scopes)->redirect($request->fullUrl()); //获取code-回调到本页
            }
    
    //        Event::fire(new WeChatUserAuthorized(session($sessionKey), $isNewSession, $account));
            
            return $next($request);
        }
```

#### 使用中间件即可
```php
Route::group(['prefix' => 'api', 'middleware' => ['wechat.oauth']],function(){
    Route::get('login','WeChatController@login');
});
```
### 微信公众号支付

#### 生成支付 jssdk
```php
public function payment(Request $request)
    {
        $type = Input::get("type");//支付类型
        $order_no = Input::get("order_no");//订单号
        if (!in_array($type,array("wechat_pay","amount","ali_pay","alipay_app"))){
            return $this->error("支付类型错误");
        }
        $is_wechat = Input::get("is_wechat","0");//是否是微信浏览器
        if (empty($order_no)) return $this->error("参数错误");
        $order = FarmOrder::where('order_no',$order_no)->where("user_id",User::getUserId())->where("pay_status",0)->first();
        if (empty($order)) return $this->error("订单未找到");
        $user_id = User::getUserId();

            //微信支付
            if($type == "wechat_pay"){
            try {
                $order->pay_id = 2;
                $user          = User::find($user_id);
                $app           = app('wechat.payment.default');
                //统一下单(预支付)
                $result        = $app->order->unify([
                    'body'         => '田园农场-支付订单',
                    'out_trade_no' => $order_no,
                    'total_fee'    => $order->total_price * 100,
//                'spbill_create_ip' => '123.12.12.123', // 可选，如不传该参数，SDK 将会自动获取相应 IP 地址
                    'notify_url'   => 'http://www.tianyuannongchang.cn/wechat/notify', // 支付结果通知网址，如果不设置则会使用配置里的默认地址
                    'trade_type'   => 'JSAPI',
                    'openid'       => $user->tync_openid,
                ]);
                //根据预支付数据 生成支付 jssdk
                $payment = Factory::payment(config('wechat.payment.default'));
                $jssdk = $payment->jssdk;
                $json = $jssdk->bridgeConfig($result['prepay_id'],false); // 返回 json 字符串，如果想返回数组，传第二个参数 false
                return $this->success($json);

            }catch (\Exception $exception){
                return $this->error($exception->getMessage());
            }
        }else{
            return $this->error("支付类型错误");
        }
    }
```

#### JS 根据返回 jssdk 唤起微信支付

```javascript

 $("#payment").click(function(){
                $.ajax({
                  url:_SERVER + "api/payment",          //上边的接口
                  type:'post',
                  dataType:'json',
                  data:{
                    type:type,
                    order_no:order_no
                  },
                  success:function(data){
                      if(data.error == 0){
                          if(type == "wechat_pay"){
                                  return  WeixinJSBridge.invoke(        //微信支付
                                      'getBrandWCPayRequest',
                                      data.msg,
                                      function(res){
                                          if(res.err_msg == "get_brand_wcpay_request:ok" ) {
                                              layer.open({
                                                  content: '恭喜您，订单支付成功'
                                                  ,btn: [ '确定']
                                                  ,yes: function(){
                                                      window.location.href="my-order.html";
                                                  }
                                              });
                                          }else{
                                              layer.open({content: '请重试' ,skin: 'msg' ,time: 1 });
                                              return false;
                                          }
                                      }
                                  );
                              
                          }
                      }else{
                          layer_msg(data.message);
                      }
                  }
                })
            })
```


#### 微信回调

```php
 public function notifyWechat(Request $request)
    {

        $app = app('wechat.payment.default');
        $response = $app->handlePaidNotify(function($message, $fail){
            // 使用通知里的 "微信支付订单号" 或者 "商户订单号" 去自己的数据库找到订单
            $order = FarmOrder::getOrderByNo($message['out_trade_no']);

            if (empty($order) || $order->pay_status == 1) { // 如果订单不存在 或者 订单已经支付过了
                return true; // 告诉微信，我已经处理完了，订单没找到，别再通知我了
            }


            if ($message['return_code'] === 'SUCCESS') { // return_code 表示通信状态，不代表支付状态
                // 用户是否支付成功
                if ($message['result_code'] === 'SUCCESS') {
                    $order->pay_time = time(); // 更新支付时间为当前时间
                    $order->pay_status = 1;

                    // 用户支付失败
                } elseif ($message['result_code'] === 'FAIL') {
                    $order->pay_status = 0;
                }
            } else {
                return $fail('通信失败，请稍后再通知我');
            }

            $order->save(); // 保存订单

            return true; // 返回处理完成
        });

        return $response;

    }
```