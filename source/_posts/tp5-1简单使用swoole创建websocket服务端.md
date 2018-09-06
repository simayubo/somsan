---
title: tp5.1简单使用swoole创建websocket服务端
date: 2018-09-06 14:42:43
tags: [swoole, websocket, php]
categories: [php]
---

## Swoole安装
### 编译安装

```bash
#下载
$ wget https://github.com/swoole/swoole-src/archive/v4.1.2.zip
#解压
$ unzip v4.1.2.zip
#进入目录
$ cd swoole-src-4.1.2
$ phpize
$ ./configure #注意，要开启ssl的话，需要加上 --enable-openssl
$ make
$ make install

```
### PECL
swoole项目已收录到PHP官方扩展库，除了手工下载编译外，还可以通过PHP官方提供的pecl命令，一键下载安装swoole
```bash
$ pecl install swoole
```

编译安装成功后，修改php.ini加入 `extension=swoole.so`
安装后可用 `php -m` 或 `phpinfo()` 来查看是否安装成功
<!--more-->

## tp5.1安装swoole Composer包
```bash
$ composer require topthink/think-swoole --ignore-platform-reqs
```
> 加上 `--ignore-platform-reqs` 表示忽略依赖检测，因为在Win下是无法安装Swoole的，Linux下开发请忽略

安装成功后在项目根目录的 `config` 目录下会多出配置文件 `swoole.php` 和 `swoole_server.php`，我们关心的是 `swoole_server.php` 这个文件

## 配置
### 编辑配置文件
打开 `swoole_server.php`
```php
<?php
// +----------------------------------------------------------------------
// | ThinkPHP [ WE CAN DO IT JUST THINK IT ]
// +----------------------------------------------------------------------
// | Copyright (c) 2006-2018 http://thinkphp.cn All rights reserved.
// +----------------------------------------------------------------------
// | Licensed ( http://www.apache.org/licenses/LICENSE-2.0 )
// +----------------------------------------------------------------------
// | Author: liu21st <liu21st@gmail.com>
// +----------------------------------------------------------------------

use think\facade\Env;

// +----------------------------------------------------------------------
// | Swoole设置 php think swoole命令行下有效
// +----------------------------------------------------------------------
return [
    // 扩展自身配置
    'host'         => '0.0.0.0', // 监听地址
    'port'         => 9501, // 监听端口
    'type'         => 'socket', // 服务类型 支持 socket http server
    'mode'         => '',
    'socket_type'  => '',
    'swoole_class' => 'app\common\library\Swoole', // 自定义服务类名称

    // 可以支持swoole的所有配置参数
    'daemonize'    => false,
    'pid_file'     => Env::get('runtime_path') . 'swoole_server.pid',
    'log_file'     => Env::get('runtime_path') . 'swoole_server.log',

    // 事件回调定义
    'onOpen'       => function ($server, $request) {
        echo "server: handshake success with fd{$request->fd}\n";
    },

    'onMessage'    => function ($server, $frame) {
        echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
        $server->push($frame->fd, "this is server");
    },

    'onRequest'    => function ($request, $response) {
        $response->end("<h1>Hello Swoole. #" . rand(1000, 9999) . "</h1>");
    },

    'onClose'      => function ($ser, $fd) {
        echo "client {$fd} closed\n";
    },
];

```
### 创建服务类
我们这里自定义了服务类 `app\common\library\Swoole` ，所以需要创建该服务类；

流程是：
1. 客户端创建链接到服务器端
2. 连接成功后服务端创建15秒后定时器，检测15秒时当前连接是否认证，没有认证直接主动断开连接
3. 认证成功后，客户端每5秒向服务端发送ping数据，客户端每5秒检测一次连接是否超过10秒无请求，如果没有则主动断开连接，如果有则检测当前认证是否还有效，身份过期或者认证失效也主动断开连接

在 `application\common\library\` 下创建 `Swoole.php` 文件，内容如下:
```php
<?php

namespace app\common\library;

use think\swoole\Server;

class Swoole extends Server
{
    protected $host = '0.0.0.0';
    protected $port = 9502;
    protected $serverType = 'socket';
    protected $sockType = SWOOLE_SOCK_TCP|SWOOLE_SSL; //SWOOLE_SSL标识开启ssl(wss)

    //Swoole server
    private $server;
    //Swoole client_id
    private $fd;
    //uid
    private $uid;

    protected $option = [
        'worker_num'	=> 4,
        'daemonize'	=> true, //true为后台运行
        'backlog'	=> 128,
        'heartbeat_check_interval' => 5, //心跳检测：每5s检测一次
        'heartbeat_idle_time' => 10, //心跳检测：超过10s未请求的连接主动关闭
        'ssl_cert_file' => '/etc/letsencrypt/live/cuuu.co/fullchain.pem', //ssl证书
        'ssl_key_file' => '/etc/letsencrypt/live/cuuu.co/privkey.pem', //ssl证书key
    ];

    /**
     * 此事件在Worker进程/Task进程启动时发生。这里创建的对象可以在进程生命周期内使用
     * @param $server
     * @param $worker_id
     */
    public function onWorkerStart($server, $worker_id){
        $this->server = $server;
        //初始化redis
        //初始化db
    }

    /**
     * 当WebSocket客户端与服务器建立连接并完成握手后会回调此函数。
     * @param $server
     * @param $request
     */
    public function onOpen($server, $request){

    }

    /**
     * 客户端收到来自于服务器端的数据时会回调此函数
     * @param $server
     * @param $fd
     * @param $from_id
     * @param $data
     */
    public function onReceive($server, $fd, $from_id, $data)
    {
        echo "接收到来自reactor:{$from_id}的数据{$data} \n";

    }

    /**
     * 有新的连接进入时，在worker进程中回调
     * @param $server
     * @param $fd
     * @param $reactorId
     */
    public function onConnect($server, $fd, $reactorId)
    {
        //连接成功时，记录当前连接的客户端ID
        $this->fd = $fd;
        swoole_timer_after(15000, function () use ($fd) {
            if (empty($this->uid)){
                //15秒后如果还未认证，主动关闭该连接
                $this->server->close($fd);
            }
        });
    }

    /**
     * 接收http请求
     * @param $request
     * @param $response
     */
    public function onRequest($request, $response){
        $response->end("<h1>Hello Swoole. #" . rand(1000, 9999) . "</h1>");
    }

    /**
     * 当服务器收到来自客户端的数据帧时会回调此函数
     * data格式 = {"type":"auth","text":"xxxxxxxxxxxxxxx"}
     * @param $server
     * @param $frame
     * @return bool|void
     */
    public function onMessage($server, $frame){

        $data = json_decode($frame->data, true);
        if (!isset($data['type'])){
            return $this->send('error', -1, '参数错误!');
        }

        switch ($data['type']){
            case 'auth':
                $this->auth($data);
                break;
            case 'ping':
                $this->ping();
                break;

            default: return false;
        }
    }

    /**
     * TCP客户端连接关闭后，在worker进程中回调此函数
     * @param $ser
     * @param $fd
     */
    public function onClose($ser, $fd){
        echo "client {$fd} closed\n";
    }

    /**
     * 发送消息
     * @param $type
     * @param $code
     * @param $message
     * @param array $data
     * @param bool $is_object
     * @return bool|void
     */
    private function send($type, $code, $message, $data = [], $is_object = true){
        if (empty($data) && $is_object){
            $data = (object)$data;
        }
        $msg = json_encode([
            'code' => $code,
            'msg' => $message,
            'type' => $type,
            'data' => $data
        ]);
        $this->server->push($this->fd, $msg);
    }

    /**
     * 用户认证
     * @param $data
     * @return bool|void
     */
    private function auth($data){
        /**
         * 用户认证
         *
         */

        /**
         * 认证后记录当前请求uid，注意swoole中是无法使用session的
         *
         *
         */
        $this->uid = 1;

        return $this->send('auth', 0, '账号认证通过');
    }


    /**
     * ping检测
     */
    private function ping(){
        if (empty($this->uid)){
            $this->send('error', -1, '请求未认证，请先认证');
            return $this->server->close($this->fd);
        }
        /**
         * 如果用户失效，则主动关闭连接
         *
         *
         */
        $is_exists = true;
        if (!$is_exists){
            $this->send('error', -1, '登陆状态失效');
            return $this->server->close($this->fd);
        }
    }
}
```

### Html文件

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body style="text-align: center">

<select id="type">
    <option value="text">text</option>
</select>
<input type="text" id="text" >
<button type="button" id="send" disabled>SEND</button>
<br>
<br>
<textarea id="label" style="min-height: 900px; width: 600px"></textarea>

<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js" type="application/javascript"></script>
<script>
    websocket = null;
    token = '';

    $(document).ready(function(){
        $("#send").attr("disabled", true);
        //登录
        login();
    });
    function print(text){
        var va = $('#label').val();
        $('#label').val(va + "\r\n" + text);
    }

    function wsConnect() {
        var wsServer = 'ws://xxx.com:9502';
        websocket = new WebSocket(wsServer);
        time = null;

        websocket.onopen = function (evt) {
            $("#send").attr("disabled", false);
            print("Connected to WebSocket server.");
            //认证
            send('auth', token);
        };
        websocket.onclose = function (evt) {
            $("#send").attr("disabled", true);
            if (time != null){
                clearTimeout(time);
            }
            print("Disconnected");
        };
        websocket.onmessage = function (evt) {
            var data = JSON.parse(evt.data);
            switch (data.type) {
                case 'auth':
                    ping();
                    break;
                case 'error':
                    print(data.msg)
                    break;
            }
            print('Retrieved data from server: ' + data.msg);
        };
        websocket.onerror = function (evt, e) {
            $("#send").attr("disabled", true);
            print('Error occured: ' + evt.data);
        };
    }

    function ping() {
        send('ping', '');
        time = setTimeout(function(){
            ping()
        }, 5000);
    }

    $('#send').click(function () {
        var txt = $('#text').val();
        var type = $('#type').val();
        send(type, txt)
    });

    function send(type = '', str = '') {
        print('send:['+type+']' + str);
        websocket.send('{"type":"' +type+ '","text":"' +str+ '"}')
    }

    function login() {
        print('登录成功');
        token = 'sssssssssssssssssssssss';
        wsConnect();
    }


</script>
</body>
</html>
```

## 最终效果图

Html效果
![lZ4.png][1]

websocket连接图
![za6.png][2]


  [1]: https://img.somsan.cc/images/2018/09/06/lZ4.png
  [2]: https://img.somsan.cc/images/2018/09/06/za6.png

