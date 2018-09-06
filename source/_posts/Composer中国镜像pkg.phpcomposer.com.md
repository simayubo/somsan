---
title: Composer中国镜像pkg.phpcomposer.com
date: 2018-08-20 13:08:11
tags: [Composer]
categories: [php]
---

## 用法：
### 有两种方式启用本镜像服务：
系统全局配置： 即将配置信息添加到 Composer 的全局配置文件 config.json 中。见“例1”
单个项目配置： 将配置信息添加到某个项目的 composer.json 文件中。见“例2”
### 例1：修改 composer 的全局配置文件（推荐方式）
打开命令行窗口（windows用户）或控制台（Linux、Mac 用户）并执行如下命令：
```bash
composer config -g -e
```
将内容替换掉
```json
{
    "repositories": [
        { "packagist": false },
        {
            "type": "composer",
            "url": "https://packagist.phpcomposer.com"
        }
    ]
}
```


<!--more-->


### 例2：修改当前项目的 composer.json 配置文件：
打开命令行窗口（windows用户）或控制台（Linux、Mac 用户），进入你的项目的根目录（也就是 composer.json 文件所在目录），执行如下命令：
```bash
composer config repo.packagist composer https://packagist.phpcomposer.com
```
上述命令将会在当前项目中的 composer.json 文件的末尾自动添加镜像的配置信息（你也可以自己手工添加）：
```json
"repositories": {
    "packagist": {
        "type": "composer",
        "url": "https://packagist.phpcomposer.com"
    }
}
```
以 laravel 项目的 composer.json 配置文件为例，执行上述命令后如下所示（注意最后几行）：
```json
{
    "name": "laravel/laravel",
    "description": "The Laravel Framework.",
    "keywords": ["framework", "laravel"],
    "license": "MIT",
    "type": "project",
    "require": {
        "php": ">=5.5.9",
        "laravel/framework": "5.2.*"
    },
    "require-dev": {
        "fzaninotto/faker": "~1.4",
        "mockery/mockery": "0.9.*",
        "phpunit/phpunit": "~4.0",
        "symfony/css-selector": "2.8.*|3.0.*",
        "symfony/dom-crawler": "2.8.*|3.0.*"
    },
    "autoload": {
        "classmap": [
            "database"
        ],
        "psr-4": {
            "App\\": "app/"
        }
    },
    "autoload-dev": {
        "classmap": [
            "tests/TestCase.php"
        ]
    },
    "scripts": {
        "post-root-package-install": [
            "php -r \"copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "php artisan key:generate"
        ],
        "post-install-cmd": [
            "php artisan clear-compiled",
            "php artisan optimize"
        ],
        "pre-update-cmd": [
            "php artisan clear-compiled"
        ],
        "post-update-cmd": [
            "php artisan optimize"
        ]
    },
    "config": {
        "preferred-install": "dist"
    },
    "repositories": {
        "packagist": {
            "type": "composer",
            "url": "https://packagist.phpcomposer.com"
        }
    }
}
```
OK，一切搞定！试一下 `composer install` 来体验飞一般的速度吧！
镜像说明：
一般情况下，安装包的数据（主要是 zip 文件）一般是从 github.com 上下载的，安装包的元数据是从 packagist.org 上下载的。
然而，由于众所周知的原因，国外的网站连接速度很慢，并且随时可能被“墙”甚至“不存在”。

“Packagist 中国全量镜像”所做的就是缓存所有安装包和元数据到国内的机房并通过国内的 CDN 进行加速，这样就不必再去向国外的网站发起请求，从而达到加速 `composer install` 以及 `composer update` 的过程，并且更加快速、稳定。因此，即使 packagist.org、github.com 发生故障（主要是连接速度太慢和被墙），你仍然可以下载、更新安装包。