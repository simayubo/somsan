---
title: 
date: 2018-09-06 11:46:42
comments: false
---

## 网站

 | 地址        | 介绍    |
 | --------   | :-----  |
|https://packagist.org/|Composer全量镜像|
|https://github.com|全球最大的开源平台|
|http://www.laruence.com/|鸟哥的博客（风雪之隅）|
|http://cdn.code.baidu.com/|百度静态资源库|
|https://segmentfault.com|segmentfault社区|
|https://laravel-china.org/|Laravel-China社区|
|https://www.oschina.net/project/lang/22/php|oschina社区|
|http://www.bootcss.com/p/font-awesome/|font-awesome图标|
|http://www.atool.org/httptest.php | 在线接口调试工具 |
|https://www.json.cn/ | Json在线解析 |
|https://travis-ci.org/ | 一个持续集成的平台 |

## Composer扩展包推荐

> 可用国内镜像：[Composer中国镜像pkg.phpcomposer.com][1]
### 通用扩展包

| 地址        | 介绍    |
 | --------   | :----- |
|[jenssegers/agent][2]| 客户端 User Agent 解析工具（基于 Mobiledetect）|
|[RoumenDamianoff/laravel-sitemap][3]| Sitemap 生成工具|
|[Chumper/Zipper][4]| ZIp 打包工具（基于 ZipArchive）|
|[SimpleSoftwareIO/simple-qrcode][5]| 二维码生成工具|
|[firebase/php-jwt][6]| PHP的Token生成方案|
|[5ini99/think-addons][7] | ThinkPHP5插件扩展|
|[PHPMailer/PHPMailer][8]  |PHP邮件发送方案|
|[PHPOffice/PHPExcel][9] | Excel处理|
|[dompdf/dompdf][10]  | PDF操作工具|
|[overtrue/wechat][11]| 可能是目前最好用的微信开发包|
|[symfony/var-dumper][12]| var_dump打印美化工具|
|[terranc/think-blade][13]|Laravel Blade模板引擎 for ThinkPHP5.0|
|[filp/whoops][14]|Whoops异常捕获|
|[fzaninotto/Streamer](https://github.com/fzaninotto/Streamer)|一个简单的面向对象流包装库|
### Laravel扩展包

 | 地址        | 介绍    |
 | --------   | :----- |
 | [barryvdh/laravel-debugbar][15] | Laravel调试工具 |
|[barryvdh/laravel-ide-helper][16]  |Laravel IDE代码助手|
|[Zizaco/entrust][17] |Laravel基于用户组的用户权限系统|
|[barryvdh/laravel-cors][18] |Laraveljs跨域问题解决方案|
|[barryvdh/laravel-dompdf][19] |Laravel PDF 操作工具（基于 dompdf ）|
|[tymondesigns/jwt-auth][20] |Laravel JWT (JSON Web Token) 用户认证机制|
|[dingo/api][21] |Laravel构建 API 服务器的完整解决方案|
|[spatie/laravel-backup][22] |Laravel数据备份工具，支持压缩，支持各种文件系统（推荐）|
|[yajra/laravel-datatables][23] |Laravel jQuery DataTables 的后端支持|
|[NauxLiu/Laravel-SendCloud][24]|Laravel的SendCloud包|

### 工具

| 地址        | 介绍    |
| --------   | :----- |
| [zircote/swagger-php](https://github.com/zircote/swagger-php) | 文档自动生成工具 |
| [taskphp](http://taskphp.github.io/) | 依据Grunt和Gulp的纯PHP任务运行器 |
| [thephpleague/omnipay](https://github.com/thephpleague/omnipay) | 多网关支付处理的框架 |
| [yansongda/pay](https://github.com/yansongda/pay) |  Alipay和WeChat的支付SDK扩展包 |
| [apache/kafka](https://github.com/apache/kafka) |  高吞吐量的分布式发布订阅消息系统 |
| [weiboad/kafka-php](https://github.com/weiboad/kafka-php) |  kafka客户端php库 |


## Linux包
### linux安装ffmpeg-php扩展

```bash
wget -c wget http://down1.chinaunix.net/distfiles/ffmpeg-0.4.9-p20051120.tar.bz2
tar -xvf ffmpeg-0.4.9-p20051120.tar.bz2
./configure --prefix=/usr/local/ffmpeg --enable-shared
make
make install
```
 
 
## 文章

| 地址        | 介绍    |
| --------   | :----- |
| http://doc.cocolian.cn/essay/ | 支付系统设计 |


  [1]: /2018/08/20/Composer中国镜像pkg.phpcomposer.com/
  [2]: https://github.com/jenssegers/agent
  [3]: https://github.com/RoumenDamianoff/laravel-sitemap
  [4]: https://github.com/Chumper/Zipper
  [5]: https://github.com/SimpleSoftwareIO/simple-qrcode
  [6]: /2018/09/03/APP接口Token生成方案php-jwt/
  [7]: https://github.com/5ini99/think-addons
  [8]: /2018/08/06/使用PHPMailer包实现PHP的邮件发送/
  [9]: https://github.com/PHPOffice/PHPExcel
  [10]: https://github.com/dompdf/dompdf/
  [11]: https://easywechat.org/
  [12]: https://github.com/symfony/var-dumper
  [13]: https://github.com/terranc/think-blade
  [14]: /2018/08/04/使用Whoops替换thinkphp5的异常获取/
  [15]: https://github.com/barryvdh/laravel-debugbar
  [16]: https://github.com/barryvdh/laravel-ide-helper
  [17]: https://github.com/Zizaco/entrust
  [18]: /2018/08/01/Laravel包barryvdh%20laravel-cors解决js跨域问题/
  [19]: https://github.com/barryvdh/laravel-dompdf
  [20]: https://github.com/tymondesigns/jwt-auth
  [21]: https://github.com/dingo/api
  [22]: https://github.com/spatie/laravel-backup
  [23]: https://github.com/yajra/laravel-datatables
  [24]: https://github.com/NauxLiu/Laravel-SendCloud