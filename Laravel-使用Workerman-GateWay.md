---
title: Laravel 使用WorkerMan(GateWay)
tags:
  - Laravel
  - WorkerMan
  - GateWay
  - PHP
keywords:
  - Laravel
  - WorkerMan
  - GateWay
  - PHP
categories:
  - 技术
abbrlink: fc208a4f
date: 2019-07-22 16:30:53
---
## 前言
记录在 Laravel 中使用 WorkerMan 或 GateWay 过程。

## 开始

### 安装

```
composer require workerman/gateway-worker
```

### 创建命令行
使用 artisan 创建一个命令行工具来启动 WorkerMan 因为 WorkerMan 只能在命令行中启动。

```
php artisan make:command WorkerMan
```

### 更改 `/app/Console/Commands/WorkerMan.php` 文件
```php
<?php

namespace App\Console\Commands;

use GatewayWorker\BusinessWorker;
use GatewayWorker\Gateway;
use GatewayWorker\Register;
use Illuminate\Console\Command;
use Workerman\Worker;

class WorkerMan extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'workman {action} {--d}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Start a Workerman server.';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        global $argv;
        $action = $this->argument('action');

        $argv[0] = 'wk';
        $argv[1] = $action;
        $argv[2] = $this->option('d') ? '-d' : '';

        $this->start();
    }

    private function start()
    {
        $this->startGateWay();
        $this->startBusinessWorker();
        $this->startRegister();
        Worker::runAll();
    }
    //BusinessWorker是运行业务逻辑的进程，BusinessWorker收到Gateway转发来的事件及请求时会默认调用Events.php中的onConnect onMessage 
    private function startBusinessWorker()
    {
        $worker                  = new BusinessWorker();
        $worker->name            = 'BusinessWorker';        //设置BusinessWorker进程的名称
        $worker->count           = 1;       //设置BusinessWorker进程的数量
        $worker->registerAddress = '127.0.0.1:2018';        //注册服务地址-向Register进程注册-内部通讯使用
        $worker->eventHandler    = \App\Workerman\Events::class;        //设置使用哪个类来处理业务,业务类至少要实现onMessage静态方法，onConnect和onClose静态方法可以不用实现
    }

    private function startGateWay()
    {
        // 证书最好是申请的证书-开启wss 打开注释
        /**
        * Workerman版本不小于3.3.7
        * PHP安装了openssl扩展 
        * 已经申请了证书（pem/crt文件及key文件）放在磁盘某个目录(位置任意)
        $context = array(
            // 更多ssl选项请参考手册 http://php.net/manual/zh/context.ssl.php
            'ssl' => array(
                // 请使用绝对路径
                'local_cert'                 => '磁盘路径/server.pem', // 也可以是crt文件
                'local_pk'                   => '磁盘路径/server.key',
                'verify_peer'               => false,
                // 'allow_self_signed' => true, //如果是自签名证书需要开启此选项
            )
        );
        */
        $gateway = new Gateway("websocket://0.0.0.0:2019");
        // 开启SSL，websocket+SSL 即wss websocket协议(端口任意，只要没有被其它程序占用就行)
        //  $gateway = new Gateway("websocket://0.0.0.0:443", $context);
        //  $gateway->transport = 'ssl';
        $gateway->name                 = 'Gateway';     //设置Gateway进程的名称，方便status命令中查看统计
        $gateway->count                = 1;     //进程的数量
        $gateway->lanIp                = '127.0.0.1';       //内网ip,多服务器分布式部署的时候需要填写真实的内网ip
        $gateway->startPort            = 2010;      //监听本机端口的起始端口
        $gateway->pingInterval         = 30;        //心跳检测时间间隔 单位：秒。如果设置为0代表不做任何心跳检测。
        $gateway->pingNotResponseLimit = 0;     //客户端连续$pingNotResponseLimit次$pingInterval时间内不发送任何数据则断开链接，并触发onClose。 如果设置为0代表客户端不用发送心跳数据

        $gateway->pingData             = '{"type":"@heart@"}';      //当需要服务端定时给客户端发送心跳数据时， $gateway->pingData设置为服务端要发送的心跳请求数据，心跳数据是任意的，只要客户端能识别即可。
        $gateway->registerAddress      = '127.0.0.1:2018';      //注册服务地址-向Register进程注册-内部通讯使用
    }

    /**
    * Gateway进程和BusinessWorker进程启动后分别向Register进程注册自己的通讯地址，Gateway进程和BusinessWorker通过Register进程得到通讯地址后，就可以建立起连接并通讯了。
    */
    private function startRegister()
    {
        new Register('text://0.0.0.0:2018');
    }
}

```

### 创建监听文件
创建 `\App\Workerman\Events.php` 文件

```php
<?php

namespace App\Workerman;


use GatewayWorker\Lib\Gateway;

class Events
{
    /**
    * 当businessWorker进程启动时触发。每个进程生命周期内都只会触发一次。
    * @param $businessWorker businessWorker进程实例
    */
    public static function onWorkerStart($businessWorker)
    {
    }
    
    /**
    * 当客户端连接上gateway进程时(TCP三次握手完毕时)触发的回调函数。
    * @param $client_id 固定为20个字符的字符串，用来全局标记一个socket连接，每个客户端连接都会被分配一个全局唯一的client_id。如果client_id对应的客
    *                   户端连接断开了，那么这个client_id也就失效了。当这个客户端再次连接到Gateway时，将会获得一个新的client_id。也就是说
    *                   client_id和客户端的socket连接生命周期是一致的。
    *                   client_id一旦被使用过，将不会被再次使用，也就是说client_id是不会重复的，即使分布式部署也不会重复。
    *                   只要有client_id，并且对应的客户端在线，就可以调用Gateway::sendToClient($client_id, $data)等方法向这个客户端发送数据。
    */
    public static function onConnect($client_id)
    {
//        Gateway::sendToCurrentClient("Your client_id is $client_id");
        Gateway::sendToClient($client_id, json_encode(array(
            'type'      => 'init',
            'client_id' => $client_id
        )));
    }

    /**
    * 当客户端连接上gateway完成websocket握手时触发的回调函数。
    * 注意：此回调只有gateway为websocket协议并且gateway没有设置onWebSocketConnect时才有效。
    * @param $client_id 固定为20个字符的字符串，用来全局标记一个socket连接，每个客户端连接都会被分配一个全局唯一的client_id。
    * @param $data websocket握手时的http头数据，包含get、server等变量
    */
    public static function onWebSocketConnect($client_id, $data)
    {
    }
    
    /**
    * 当客户端发来数据(Gateway进程收到数据)后触发的回调函数
    * @param $client_id 全局唯一的客户端socket连接标识
    * @param $message 完整的客户端请求数据，数据类型取决于Gateway所使用协议的decode方法返的回值类型
    */
    public static function onMessage($client_id, $message)
    {

    }
    
    /**
    * 客户端与Gateway进程的连接断开时触发。不管是客户端主动断开还是服务端主动断开，都会触发这个回调。一般在这里做一些数据清理工作。
    * 注意：onClose回调里无法使用Gateway::getSession()来获得当前用户的session数据，但是仍然可以使用$_SESSION变量获得。
    * 注意：onClose回调里无法使用Gateway::getUidByClientId()接口来获得uid，解决办法是在Gateway::bindUid()时记录一个$_SESSION['uid']，onClose的时候用$_SESSION['uid']来获得uid。
    * 注意：断网断电等极端情况可能无法及时触发onClose回调，因为这种情况客户端来不及给服务端发送断开连接的包(fin包)，服务端就无法得知连接已经断开。检测这种极端情况需要心跳检测，并且必须设置$gateway->pingNotResponseLimit>0。这种断网断电的极端情况onClose将被延迟触发，延迟时间为小于$gateway->pingInterval*$gateway->pingNotResponseLimit秒，如果$gateway->pingInterval 和 $gateway->pingNotResponseLimit 中任何一个为0，则可能会无限延迟。
    * @param $client_id 全局唯一的client_id
    */
    public static function onClose($client_id)
    {
    }
}
```

### 启动 WorkerMan 服务端
在命令行里面执行，支持的命令大概有 start|stop|restart，其中 -d 的意思是 daemon 模式。
```
php artisan workman start -d
```

### 测试
在浏览器 F12 打开调试模式，在 Console 里输入
```javascript
// 打开一个WebSocket:
var ws = new WebSocket('ws://localhost:2019');
// 响应onmessage事件:
ws.onmessage = function(msg) {
                    console.log(msg); 
               };
// 给服务器发送一个字符串:
ws.send('Hello!');
```