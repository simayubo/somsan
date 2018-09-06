---
title: 使用Whoops替换thinkphp5的异常获取
date: 2018-08-04 16:29:11
tags: [php]
categories: [php]
---

## 介绍
Whoops是一个很漂亮的异常捕获工具包，tp5自带的自己用不习惯，就换了这个东西，Github: [https://github.com/filp/whoops][1]，先放个预览图：

![QQ截图20170811114522.png][2]


<!--more-->


## 安装方法：

```php
composer require filp/whoops
```

### 配置Thinkphp5

1. 在你的项目里新建一个文件，如：我新建文件 `application/common/exception/Http.php`
2. 在文件里写入下面代码：

    ```php 
 <?php
    namespace app\common\exception;
    use Exception;
    use think\exception\Handle;

    class Http extends Handle
    {

        public function render(Exception $e)
        {
            if (version_compare(PHP_VERSION, '5.5.9', '<')){  //这里判断如果php版本比较低，就默认用系统的，当然你也可以安装版本较低的Whoops
                return parent::render($e);
            }else{
                $whoops = new \Whoops\Run;

                if (request()->isAjax()){ //如果是ajax请求，就返回json数据
                    $whoops->pushHandler(new \Whoops\Handler\JsonResponseHandler);
                    return  $whoops->handleException($e);
                }else{
                    //否则返回html数据
                    $whoops->pushHandler(new \Whoops\Handler\PrettyPageHandler);
                    return  $whoops->handleException($e);
                }
            }
        }
    }
    ```

3. 打开配置文件`application/config.php`，找到`exception_handle`配置，修改为：
 ```php
 \\app\\common\\exception\\Http
 ```

大功告成！


  [1]: https://github.com/filp/whoops
  [2]: https://img.somsan.cc/images/2018/09/06/atm.png