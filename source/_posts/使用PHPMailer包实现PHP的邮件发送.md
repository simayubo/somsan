---
title: 使用PHPMailer包实现PHP的邮件发送
tags:
  - php
  - Composer
categories:
  - php
abbrlink: '50155940'
date: 2018-08-06 12:18:11
---

## 介绍

- 可能是世界上最流行的从PHP发送电子邮件的代码！
- 许多开源项目使用：WordPress，Drupal，1CRM，SugarCRM，Yii，Joomla！还有很多
- 集成SMTP支持 - 无需本地邮件服务器即可发送
- 发送多个TOs，CC，BCC和REPLY TO的电子邮件
- 不阅读HTML电子邮件的邮件客户端的多部分/替代电子邮件
- 支持UTF-8内容和8bit，base64，二进制和可引用打印编码
- 使用LOGIN，PLAIN，NTLM，CRAM-MD5和Google的XOAUTH2机制通过SSL和TLS传输进行SMTP验证
- 47种语言的错误信息！
- DKIM和S / MIME签名支持
- 兼容PHP 5.0及更高版本
- 等等

## 地址
Packagist：[https://packagist.org/packages/phpmailer/phpmailer][1]
Github：[https://github.com/PHPMailer/PHPMailer][2]


<!--more-->


## 安装方法
安装包
```bash
composer require phpmailer/phpmailer
```
首先编写一个函数
```php
/**
 * 系统邮件发送函数
 * @param string $tomail 接收邮件者邮箱
 * @param string $name 接收邮件者名称
 * @param string $subject 邮件主题
 * @param string $body 邮件内容
 * @param string $attachment 附件列表
 * @return boolean
 */
function send_mail($tomail_array, $attachment = null)
{
    $mail = new \PHPMailer();           //实例化PHPMailer对象
    $mail->CharSet = 'UTF-8';           //设定邮件编码，默认ISO-8859-1，如果发中文此项必须设置，否则乱码
    $mail->IsSMTP();                    // 设定使用SMTP服务
    $mail->SMTPDebug = 0;               // SMTP调试功能 0=关闭 1 = 错误和消息 2 = 消息
    $mail->SMTPAuth = true;             // 启用 SMTP 验证功能
    $mail->SMTPSecure = 'ssl';          // 使用安全协议
    $mail->Host = "smtp.qq.com"; // SMTP 服务器
    $mail->Port = 465;                  // SMTP服务器的端口号
    $mail->Username = "xxxxxxx";    // SMTP服务器用户名
    $mail->Password = "xxxxxxx";     // SMTP服务器密码

    $mail->SetFrom('xxxxx@qq.com', 'xxxx');
    $replyEmail = '';                   //留空则为发件人EMAIL
    $replyName = '';                    //回复名称（留空则为发件人名称）
    $mail->AddReplyTo($replyEmail, $replyName);
    $mail->Subject = $tomail_array['subject'];  //标题
    $mail->MsgHTML($tomail_array['content']);  //内容
    $mail->AddAddress($tomail_array['toemail'], $tomail_array['name']); //目标邮件

    if (is_array($attachment)) { // 添加附件
        foreach ($attachment as $file) {
            is_file($file) && $mail->AddAttachment($file);
        }
    }

    return $mail->Send() ? true : $mail->ErrorInfo;
}
```

然后在你需要的地方调用
```php
$item = [
   'toemail' => 'xxxx@qq.com',   //目标邮件地址
   'name' => '', //名称
   'subject' => '', //邮件标题
   'content' => ''  //邮件内容，支持html
];
$res = send_mail($item)
dd($res);
```

除此之外，还可以群发，发附件等等，详情请到github中查看！


  [1]: https://packagist.org/packages/phpmailer/phpmailer
  [2]: https://github.com/PHPMailer/PHPMailer