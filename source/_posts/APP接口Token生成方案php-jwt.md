---
title: APP接口Token生成方案php-jwt
date: 2018-09-03 08:12:11
tags: [php-jwt, Token]
categories: [php]
---

## 安装方法
```
composer require firebase/php-jwt
```
## 使用方法
#### 首先引入JWT
```php
use Firebase\JWT\JWT;
```
#### 用户登录后生成token
```php
$user = User::where(['phone' => $data['phone'], 'password' => md5($data['password'])])->find();

if ($user){
   $token = $this->getToken($user);
   $this->returnMsg(true, '登录成功', 0, ['token' => $token]);
}else{
   $this->returnMsg(false, '账号或密码错误');
}
```


<!--more-->


#### 生成Token方法
```php
/**
 * 根据用户信息获取token
 * @param $user
 * @return string
 */
protected function getToken($user){

    $tokenId    = base64_encode(mcrypt_create_iv(32));
    $issuedAt   = time();
    $notBefore  = $issuedAt;
    $expire     = $notBefore + config('jwt.expire_time');
    $serverName = '127.0.0.1';

    $data = [
        'iat'  => $issuedAt, //jwt的签发时间
        'jti'  => $tokenId, //jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
        'iss'  => $serverName, //jwt签发者
        'nbf'  => $notBefore, //定义在什么时间之前，该jwt都是不可用的.
        'exp'  => $expire, //jwt的过期时间，这个过期时间必须要大于签发时间
        'data' => [ //附带数据，这里用来存储用户的id
            'userId'   => $user->id
        ]
    ];

    $token = JWT::encode($data, '这里为自定义秘钥（zzz）');

    return $token; //返回token
}

```
#### 接受token，并且获取用户信息

app请求时，可以把上面生成的token，放在请求头，get参数，post参数等，我们这里是放在了header，例如
```
authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE0OTQzMTQ3NjQsImp0aSI6IjFQQ0UyejlDRmdGeXZUVHNSRXZESXkzT3k3VGcyMHo0TDNvOGhDMjRzTkE9IiwiaXNzIjoiMTI3LjAuMC4xIiwibmJmIjoxNDk0MzE0NzY0LCJleHAiOjE0OTU2MTA3NjQsImRhdGEiOnsidXNlcklkIjo0fX0.ZBr-zABH8-eby_Wut1oLKJWk27Cfm_g5_WV560Gwmpg
```

```php
/**
 * 通过Token获取用户信息
 * @param $token
 * @return array|false|\PDOStatement|string|\think\Model
 */
protected function getUser($token = ''){

    if ($token == ''){
        if (empty($_SERVER['REDIRECT_HTTP_AUTHORIZATION'])){
            $this->returnMsg(false, 'Token缺失', 10001, [], 401);
        }else{
            $token = $_SERVER['REDIRECT_HTTP_AUTHORIZATION'];  //从header中接收token
        }
    }

    try
    {
        //从token中取出数据
        $jwt_data = JWT::decode($token, '这里是秘钥，自己随便填写（zzz）', array('HS256'));
        //获取用户信息
        $user_info = User::find($jwt_data->data->userId); 

        if (empty($user_info))  $this->returnMsg(false, '用户不存在', 10004, [], 401);
        if ($user_info->status == -1) $this->returnMsg(false, '用户已被禁用', 10005, [], 401);

        return $user_info;

    }catch(\Exception $e){
        //通过捕获异常来判断app是否需要重新登陆获取token
        if ($e->getMessage() == 'Expired token'){

            $this->returnMsg(false, 'Token过期', 10003, [], 401); //token过期
        }else{
            $this->returnMsg(false, 'Token错误', 10002, [], 401);
        }
        //后面还有其他错误，这里不进行一一说明
    }
}
```

好了，现在给予token的app保持用户状态已经基本可以用了，需要获取用户信息的时候，直接调用 `getUser()`方法，该方法会自动判断用户状态，如果没有登录或者没有请求token，会自动反馈给APP，APP自动跳转到登录页面。
