<h1 align="center">Easy SMS</h1>

<p align="center">:calling: 一款满足你的多种发送需求的短信发送组件</p>

<p align="center">
<a href="https://travis-ci.org/overtrue/easy-sms"><img src="https://travis-ci.org/overtrue/easy-sms.svg?branch=master" alt="Build Status"></a>
<a href="https://packagist.org/packages/overtrue/easy-sms"><img src="https://poser.pugx.org/overtrue/easy-sms/v/stable.svg" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/overtrue/easy-sms"><img src="https://poser.pugx.org/overtrue/easy-sms/v/unstable.svg" alt="Latest Unstable Version"></a>
<a href="https://scrutinizer-ci.com/g/overtrue/easy-sms/?branch=master"><img src="https://scrutinizer-ci.com/g/overtrue/easy-sms/badges/quality-score.png?b=master" alt="Scrutinizer Code Quality"></a>
<a href="https://scrutinizer-ci.com/g/overtrue/easy-sms/?branch=master"><img src="https://scrutinizer-ci.com/g/overtrue/easy-sms/badges/coverage.png?b=master" alt="Code Coverage"></a>
<a href="https://packagist.org/packages/overtrue/easy-sms"><img src="https://poser.pugx.org/overtrue/easy-sms/downloads" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/overtrue/easy-sms"><img src="https://poser.pugx.org/overtrue/easy-sms/license" alt="License"></a>
</p>


## 环境需求

- PHP >= 5.6

## 安装

```shell
$ composer require "overtrue/easy-sms"
```

## 使用

```php
use Overtrue\EasySms\EasySms;

$config = [
    // HTTP 请求的超时时间（秒）
    'timeout' => 5.0,
    
    // 默认发送配置
    'default' => [
        // 网关调用策略，默认：顺序调用
        'strategy' => \Overtrue\EasySms\Strategies\OrderStrategy::class
        
        // 默认可用的发送网关
        'gateways' => [
            'yunpian', 'alidayu',
        ],
    ],
    // 可用的网关配置
    'gateways' => [
        'errorlog' => [
            'file' => '/tmp/easy-sms.log',
        ],
        'yunpian' => [
            'api_key' => '824f0ff2f71cab52936axxxxxxxxxx',
        ],
        'alidayu' => [
            //...
        ],
    ],
];

$easySms = new EasySms($config);

$easySms->send(13188888888, 
    'content'  => '您的验证码为: 6379', 
    'template' => 'SMS_001', 
    'data' => [ 
        'code' => 6379
    ],
 ]);
```

## 短信内容

由于使用多网关发送，所以一条短信要支持多平台发送，每家的发送方式不一样，但是我们抽象定义了以下公用属性：

- `content` 文字内容，使用在像云片类似的以文字内容发送的平台
- `template` 模板 ID，使用在以模板ID来发送短信的平台
- `data`  模板变量，使用在以模板ID来发送短信的平台

所以，在使用过程中你可以根据所要使用的平台定义发送的内容。

## 发送网关

默认使用 `default` 中的设置来发送，如果某一条短信你想要覆盖默认的设置。在 `send` 方法中使用第三个参数即可：

```php
$easySms->send(13188888888, 
    'content'  => '您的验证码为: 6379', 
    'template' => 'SMS_001', 
    'data' => [ 
        'code' => 6379
    ],
 ], ['yunpian', 'juhe']); // 这里的网关配置将会覆盖全局默认值
```

## 返回值

由于使用多网关发送，所以返回值为一个数组，结构如下：
```php
[
    'yunpian' => [
        'status' => 'success',
        'result' => [...] // 平台返回值
    ],
    'juhe' => [
        'status' => 'erred',
        'exception' => \Overtrue\EasySms\Exceptions\GatewayErrorException 对象
    ],
    //...
]
```

## 定义短信

你可以根本发送场景的不同，定义不同的短信类，从而实现一处定义多处调用，你可以继承 `Overtrue\EasySms\Message` 来定义短信模型：

```php
<?php

use Overtrue\EasySms\Message;
use Overtrue\EasySms\Strategies\OrderStrategy;

class OrderPaidMessage extends Messeage
{
    protected $order;
    protected $strategy = OrderStrategy::class;           // 定义本短信的网关使用策略，覆盖全局配置中的 `default.strategy`
    protected $gateways = ['alidayu', 'yunpian', 'juhe']; // 定义本短信的适用平台，覆盖全局配置中的 `default.gateways`

    public function __construct($order)
    {
        $this->order = $order;
    }
        
    // 定义直接使用内容发送平台的内容
    public function getContent()
    {
        return sprintf('您的订单:%s, 已经完成付款', $this->order->no);    
    }
    
    // 定义使用模板发送方式平台所需要的模板 ID
    public function getTemplate()
    {
        return 'SMS_003'; 
    }
        
    // 模板参数
    public function getData()
    {
        return [
            'order_no' => $this->order->no    
        ];    
    }
}
```

> 更多自定义方式请参考：[`Overtrue\EasySms\Message`](Overtrue\EasySms\Message;)

发送自定义短信：

```php
$order = ...;
$message = new OrderPaidMessage($order);

$easySms->send(13188888888, $message);
```

## 平台支持

- [云片](https://github.com/overtrue/easy-sms/wiki/GateWays---Yunpian)
- [Submail](https://github.com/overtrue/easy-sms/wiki/GateWays---Submail)
- [螺丝帽](https://github.com/overtrue/easy-sms/wiki/GateWays---Luosimao)
- [阿里大鱼](https://github.com/overtrue/easy-sms/wiki/GateWays---AliDayu)
- [容联云通讯](https://github.com/overtrue/easy-sms/wiki/GateWays---Yuntongxun)
- [互亿无线](https://github.com/overtrue/easy-sms/wiki/GateWays---Huyi)
- [聚合数据](https://github.com/overtrue/easy-sms/wiki/GateWays---Juhe)
- SendCloud

## License

MIT
