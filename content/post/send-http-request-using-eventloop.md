---
title: "Send Http Request Using Eventloop"
date: 2019-07-31T16:31:04+08:00
draft: false
tags: ["PHP"]
---

最近公司发起了一个黑客马拉松大赛，要求发起N个http请求时常不能叠加，如请求两个接口每个接口耗时1秒，则响应应该在1.0x秒附近，而不是2秒或更久

分析了一下需求，理论上使用libevent可以实现，考虑到直接使用PHP的event库可能比较复杂，所以使用了react的http-client可以实现

## 安装

```shell
composer require react/http-client:^0.5.9
```

## 测试程序

```php
<?php
header('Content-Type: text/html; charset=utf-8');
require __DIR__ . '/../vendor/autoload.php';
$loop = React\EventLoop\Factory::create();
$client = new React\HttpClient\Client($loop);

for ($i = 0; $i < 3; $i++) {
    $request = $client->request('GET', 'http://local-api.example.com/?q=my' . $i);
    $request->on('response', function ($response) {
        $response->on('data', function ($chunk) {
            echo $chunk . "\n";
        });
        $response->on('end', function () {
            echo '';
        });
    });
    $request->on('error', function (\Exception $e) {
        echo $e;
    });
    $request->end();
}

$s = microtime(true);
$loop->run();
echo microtime(true) - $s;
exit;
```

### 输出结果
my0 my2 my1 1.0807709693909

通过结果得知三个接口用时1.08秒






