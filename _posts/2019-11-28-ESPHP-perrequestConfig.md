---
layout: post
title:  ElasticSearch-PHP单请求配置
date:   2019-11-28
categories: ElasticSearch ElasticSearch-PHP 单请求配置
permalink : ElasticSearch/perrequestConfig.html
description: Elasticsearch-PHP 单请求配置
---

##### 请求识别

给请求增加识别号，用于搜索慢查询日志或发现正在运行的任务。

```php
$client = ClientBuilder::create()->build();

$params = [
    'index' => 'test',
    'id'    => 1,
    'client' => [
        'opaqueId' => 'app17@dc06.eu_user1234', 
    ]
];
$response = $client->get($params);
```

##### 忽略异常

有时发生错误需要得到响应正文而不需要抛出异常的时候，就可以使用忽略异常，ES的异常与HTTP状态码对应

```php
$client = ClientBuilder::create()->build();

$params = [
    'index'  => 'test_missing',
    'id'     => 1,
    'client' => [ 'ignore' => 404 ] 
];
echo $client->get($params);

//{"_index":"test_missing","_type":"_doc","_id":"1","found":false}
```

忽略404

```php
$client = ClientBuilder::create()->build();

$params = [
    'index'  => 'test_missing',
    'client' => [ 'ignore' => [400, 404] ] 
];
echo $client->get($params);

//> No handler found for uri [/test_missing/test/] and method [GET]
```

//忽略400 404 **注意：响应体有可能不是json**

##### 自定义请求参数

ES的请求参数都在预定义的白名单中，如果需要提供自定义的查询参数，例如第三方插件或代理的身份验证令牌，则需要自定义请求参数,以绕过白名单验证

```php
$client = ClientBuilder::create()->build();

$params = [
    'index'  => 'test',
    'id'     => 1,
    'parent' => 'abc',              // white-listed Elasticsearch parameter
    'client' => [
        'custom' => [
            'customToken' => 'abc', // user-defined, not white listed, not checked
            'otherToken'  => 123
        ]
    ]
];
$exists = $client->exists($params);
```

**查看详细响应**

默认情况下，客户端只返回响应体，如果需要更详细的信息（transfer, headers, status codes等等）,可以通过配置verbose参数来实现。

不带verbose参数的默认返回体

```php
$client = ClientBuilder::create()->build();

$params = [
    'index' => 'test',
    'id'    => 1
];
$response = $client->get($params);
print_r($response);


Array
(
    [_index] => test
    [_type] => _doc
    [_id] => 1
    [_version] => 1
    [found] => 1
    [_source] => Array
        (
            [field] => value
        )

)
```

带verbose参数的返回信息

```php
$client = ClientBuilder::create()->build();

$params = [
    'index'  => 'test',
    'id'     => 1,
    'client' => [
        'verbose' => true
    ]
];
$response = $client->get($params);
print_r($response);
Array
(
    [transfer_stats] => Array
        (
            [url] => http://127.0.0.1:9200/test/test/1
            [content_type] => application/json; charset=UTF-8
            [http_code] => 200
            [header_size] => 86
            [request_size] => 51
            [filetime] => -1
            [ssl_verify_result] => 0
            [redirect_count] => 0
            [total_time] => 0.00289
            [namelookup_time] => 9.7E-5
            [connect_time] => 0.000265
            [pretransfer_time] => 0.000322
            [size_upload] => 0
            [size_download] => 96
            [speed_download] => 33217
            [speed_upload] => 0
            [download_content_length] => 96
            [upload_content_length] => -1
            [starttransfer_time] => 0.002796
            [redirect_time] => 0
            [redirect_url] =>
            [primary_ip] => 127.0.0.1
            [certinfo] => Array
                (
                )

            [primary_port] => 9200
            [local_ip] => 127.0.0.1
            [local_port] => 62971
        )

    [curl] => Array
        (
            [error] =>
            [errno] => 0
        )

    [effective_url] => http://127.0.0.1:9200/test/test/1
    [headers] => Array
        (
            [Content-Type] => Array
                (
                    [0] => application/json; charset=UTF-8
                )

            [Content-Length] => Array
                (
                    [0] => 96
                )

        )

    [status] => 200
    [reason] => OK
    [body] => Array
        (
            [_index] => test
            [_type] => _doc
            [_id] => 1
            [_version] => 1
            [found] => 1
            [_source] => Array
                (
                    [field] => value
                )
        )
)
```

##### curl 超时设置

通过 `timeout` 和 `connect_timeout` 参数可以配置每个请求的 Curl 超时时间。这个配置控制客户端的超时时间。 `connect_timeout` 参数控制在连接阶段完成前的 curl 的等待时间。而 `timeout` 参数则控制整个请求完成前最多需要等待多长时间。

如果超过请求时间，curl 将会关闭链接并返回一个错误。两个参数都要用秒作为单位指定.

备注：客户端超时不意味着 Elasticsearch 会中止请求. Elasticsearch 将会继续执行请求直到请求完成。在慢查询或批量查询的情况下，操作会继续在后台执行，对于客户端来这些动作说是隐蔽的。 如果你的客户端在超时后立即终止了连接，然后又发出了另一个请求，由于客户端没有服务器端的 "背压" 机制，会导致服务端过载。遇到这种情况，你会发现连接池队列会逐渐增大，当超出队列的负荷时，Elasticsearch 会发送 `EsRejectedExecutionException` 异常。

```php
$client = ClientBuilder::create()->build();

$params = [
    'index'  => 'test',
    'id'     => 1,
    'client' => [
        'timeout' => 10,        // ten second timeout
        'connect_timeout' => 10
    ]
];
$response = $client->get($params);
```

##### 开启未来模式

客户端支持异步批量处理请求，使用future参数开启

```php
$client = ClientBuilder::create()->build();

$params = [
    'index'  => 'test',
    'id'     => 1,
    'client' => [
        'future' => 'lazy'
    ]
];
$future = $client->get($params);
$results = $future->wait();  
```

支持两个参数`true` or `lazy`，详情请见[未来模式](future.html)

##### SSL 加密

```php
$client = ClientBuilder::create()->build();

$params = [
    'index'  => 'test',
    'id'     => 1,
    'client' => [
        'verify' => 'path/to/cacert.pem'      //Use a self-signed certificate
    ]
];
$result = $client->get($params);
```

详情请见[授权与SSL加密](auth.html)

