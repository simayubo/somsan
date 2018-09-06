---
title: Paypal预授权支付
tags:
  - 支付
  - php
  - paypal
categories:
  - php
abbrlink: 91e1b962
date: 2018-08-07 10:01:11
---

因为paypal退款也有手续费，所以可以使用预授权支付，订单产生后，你可以检验订单是否存在欺诈行为，如果存在，则取消订单付款授权，因为你并没有收取用户费用，所以这样不会产生手续费！

## 安装composer包
```
composer require "paypal/rest-api-sdk-php"
```
## 申请Paypal应用
- 创建paypal开发者账号：https://developer.paypal.com/
- 创建应用：
 找到`REST API apps`,然后常见应用`Create App`:

<!--more-->

![QQ截图20170905230622.jpg][1]
- 创建成功后，你可以拿到`Client ID`和`Secret`，（这里分sandbox（测试）和live（正式））

![QQ截图20170905230925.jpg][2]

## 代码
### 信息收集

```php
/**
 * paypal支付
 */
public function paypal(){

    try{
        //预授权支付
        $apiContext = new ApiContext(
            new OAuthTokenCredential(
                $this->config['PAYPAL_APPID'], //Client ID
                $this->config['PAYPAL_APPKEY']  //Secret
            )
        );
        $apiContext->setConfig(['mode' => 'live']); //如果使用正式环境，请加上这个

    }catch (\Exception $exception){
        $this->error('系统错误：'.$exception->getMessage(), url('home/user/order'));
    }

    $payer = new Payer();
    $payer->setPaymentMethod("paypal");  //支付方式 "credit_card", "bank", "paypal", "pay_upon_invoice", "carrier", "alternate_payment"

    $item1 = new Item();
    $item1->setName('游戏充值') //商品名
        ->setCurrency('USD')  //支付货币
        ->setQuantity(1) //数量
        ->setPrice(18.60); //金额

    $itemList = new ItemList();
    $itemList->setItems(array($item1)); //商品，这里可以传入多个array($item1, $item2)

    $details = new Details();

    $amount = new Amount();
    $amount->setCurrency('USD') //货币
        ->setTotal(18.60) //总金额
        ->setDetails($details);

    $transaction = new Transaction();
    $transaction->setAmount($amount)
        ->setItemList($itemList)
        ->setInvoiceNumber('KT123456789'); //订单号

    $baseUrl = getBaseUrl();
    $redirectUrls = new RedirectUrls();
    $redirectUrls->setReturnUrl("$baseUrl/home/pay/auth_return?success=true")  //授权登录后跳转地址
        ->setCancelUrl("$baseUrl/home/pay/suc/id/{$order_info['id']}?success=false");  //取消支付跳转地址

    $payment = new Payment();
    $payment->setIntent("authorize")
        ->setPayer($payer)
        ->setRedirectUrls($redirectUrls)
        ->setTransactions(array($transaction));

    $request = clone $payment;

    try {
        $payment->create($apiContext);
    } catch (\Exception $ex) {
        $this->error('错误：'.$ex->getMessage(), url('suc', ['id' => $order_info['id']]).'?success=false');
    }

    $approvalUrl = $payment->getApprovalLink();

    $this->redirect($approvalUrl);
}

```

### 信息确认

```php
/**
 * paypal预授权回调(信息确认)
 */
public function auth_return(){
    try{
        //预授权支付
        $apiContext = new ApiContext(
            new OAuthTokenCredential(
                $this->config['PAYPAL_APPID'], //Client ID
                $this->config['PAYPAL_APPKEY']  //Secret
            )
        );
        $apiContext->setConfig(['mode' => 'live']);

    }catch (\Exception $exception){
        $this->error('系统错误：'.$exception->getMessage(), url('home/user/order'));
    }

    if (isset($_GET['paymentId']) && isset($_GET['token']) && isset($_GET['PayerID'])) {

        $paymentId = $_GET['paymentId'];
        $payment = Payment::get($paymentId, $apiContext);

        $transaction = $payment->transactions[0]->toArray();

        $execution = new PaymentExecution();
        $execution->setPayerId($_GET['PayerID']);

        $transaction = new Transaction();
        $amount = new Amount();
        $details = new Details();

        $amount->setCurrency('USD'); //货币
        $amount->setTotal(18.60); //总金额
        $amount->setDetails($details);
        $transaction->setAmount($amount);

        $execution->addTransaction($transaction);

        try {
            $result = $payment->execute($execution, $apiContext);
            if ($result){
                $this->success('支付成功');
            }else{
                $this->error('支付失败');
            }
        } catch (Exception $ex) {
            dd($ex->getMessage());
        }
    }else{
        $this->error('参数错误');
    }
}

```

### IPN回调

```php
/**
 * paypal回调(使用的是ipn，注意这里没有用webhook，请在商户号中配置ipn地址，且配置支付便便为utf-8)
 * @return bool
 */
public function paypal_return()
{
    if (!request()->isPost()) die();
    $post = request()->post();
  
    $order_sn = $post['invoice'];

    //$url = 'https://www.paypal.com/cgi-bin/webscr'; //正式live环境
    $url = 'https://www.sandbox.paypal.com/cgi-bin/webscr'; //sandbox测试环境

    $data['cmd'] = '_notify-validate';
    foreach ($post as $key => $item)  $data[$key] = ($item);  //如果验证失败，请尝试加上urlencode，或者是编码错误

    $res = $this->http($url, $data, 'POST');

    if (!empty($res)){
        if (strcmp($res, "VERIFIED") == 0) {
            if ($post['payment_status'] == 'Completed') {
                //付款完成
                
            }elseif($post['payment_status'] == 'Refunded'){
                //退款
                
            }elseif($post['payment_status'] == 'Voided'){
                //取消支付授权（作废订单）
                
            }elseif($post['payment_status'] == 'Pending'){
                //订单审核
                
            }

        }else if (strcmp ($res, "INVALID") == 0) {
            //未通过认证，有可能是编码错误或非法的 POST 信息
        }
    }else{
       //未通过认证，有可能是编码错误或非法的 POST 信息
    }
    echo "fail";
}

```

### curl助手函数：
```php
/**
 * 模拟提交参数，支持https提交 可用于各类api请求
 * @param string $url ： 提交的地址
 * @param array $data :POST数组
 * @param string $method : POST/GET，默认GET方式
 * @return mixed
 */
function http($url, $data='', $method='GET', $authorization = ''){
    $headers = array(
        'Authorization: Bearer '.$authorization,
        'Content-Type' => 'x-www-form-urlencoded'
      );

    $curl = curl_init(); // 启动一个CURL会话
    curl_setopt($curl, CURLOPT_URL, $url); // 要访问的地址
    curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false); // 对认证证书来源的检查
    curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false); // 从证书中检查SSL加密算法是否存在
    curl_setopt($curl, CURLOPT_USERAGENT, $_SERVER['HTTP_USER_AGENT']); // 模拟用户使用的浏览器
    curl_setopt($curl, CURLOPT_FOLLOWLOCATION, 1); // 使用自动跳转
    curl_setopt($curl, CURLOPT_AUTOREFERER, 1); // 自动设置Referer
    if (!empty($authorization)){
        curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
    }
    if($method=='POST'){
        curl_setopt($curl, CURLOPT_POST, 1); // 发送一个常规的Post请求
        if ($data != ''){
            curl_setopt($curl, CURLOPT_POSTFIELDS, $data); // Post提交的数据包
        }
    }
    curl_setopt($curl, CURLOPT_TIMEOUT, 30); // 设置超时限制防止死循环
    curl_setopt($curl, CURLOPT_HEADER, 0); // 显示返回的Header区域内容
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // 获取的信息以文件流的形式返回
    $tmpInfo = curl_exec($curl); // 执行操作
    curl_close($curl); // 关闭CURL会话
    return $tmpInfo; // 返回数据
}

```

## 其它
1. 注意商户编码设为utf-8，不然ipn验证失败，在‘用户信息-用户信息与设置-销售工具-PayPal按钮语言编码-更多选项-否，请使用-选择 UTF-8’
2. 出现下面情况（**抱歉，我们现在无法完成您的购物 请返回到商家网站并选择其他付款方式。**），请检查你的商户是否支持你支付账号所在的国家区域

 ![QQ截图20170905233450.jpg][3]

3. ipn设置"用户信息-销售工具-即时付款通知-选择ipn设置"

注：做的时候可能会出现很多问题，要注意的基本就这些，如果有其他问题可以留言告诉我！
还有就是不要忘记了引入命名空间哦！
```php
use PayPal\Api\Amount;
use PayPal\Api\Details;
use PayPal\Api\Item;
use PayPal\Api\ItemList;
use PayPal\Api\Payer;
use PayPal\Api\Payment;
use PayPal\Api\PaymentExecution;
use PayPal\Api\RedirectUrls;
use PayPal\Api\Transaction;
use PayPal\Auth\OAuthTokenCredential;
use PayPal\Rest\ApiContext;
```


  [1]: https://simayubocc.oss-cn-hangzhou.aliyuncs.com/img/2017/09/3006909208.jpg
  [2]: https://simayubocc.oss-cn-hangzhou.aliyuncs.com/img/2017/09/2749984631.jpg
  [3]: https://simayubocc.oss-cn-hangzhou.aliyuncs.com/img/2017/09/314671106.jpg