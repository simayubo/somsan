---
title: Laravel包barryvdh laravel-cors解决js跨域问题
date: 2018-08-01 12:12:11
tags: [php, laravel, Composer]
categories: [php]
---

> github地址：https://github.com/barryvdh/laravel-cors

## 安装
首先引入包：
```json
$ composer require barryvdh/laravel-cors
```
## 生成配置文件
```bash
php artisan vendor:publish --provider="Barryvdh\Cors\ServiceProvider"
```
代码如下
```php
return [
     /*
     |--------------------------------------------------------------------------
     | Laravel CORS
     |--------------------------------------------------------------------------
     |

     | allowedOrigins, allowedHeaders and allowedMethods can be set to array('*')
     | to accept any value.
     |
     */
    'supportsCredentials' => false,
    'allowedOrigins' => ['*'],
    'allowedHeaders' => ['*'], // ex : ['Content-Type', 'Accept']
    'allowedMethods' => ['*'], // ex: ['GET', 'POST', 'PUT',  'DELETE']
    'exposedHeaders' => [],
    'maxAge' => 0,
    'hosts' => [],
]
```
在app.php中添加：
```php
Barryvdh\Cors\ServiceProvider::class,
```
## 使用
在路由中使用
```php
Route::group(['middleware' => 'cors'], function(Router $router){
    $router->get('api', 'ApiController@index');
});
```
在中间件中使用
```php
protected $middleware = [
    ....
    \Barryvdh\Cors\HandleCors::class
];
```
