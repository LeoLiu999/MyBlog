---
layout: post
title:  ElasticSearch-PHP配置
date:   2019-11-28
categories: ElasticSearch ElasticSearch-PHP install
permalink : ElasticSearch/configure.html
description: Elasticsearch-PHP类库配置
---

##### hosts配置 

```php
//内联主机配置

$hosts = [
    '127.0.0.1:9200'
];
//多节点配置
$hosts = [
    '192.168.1.1:9200',         // IP + Port
    '192.168.1.2',              // Just IP
    'mydomain.server.com:9201', // Domain + Port
    'mydomain2.server.com',     // Just Domain
    'https://localhost',        // SSL to localhost
    'https://192.168.1.3:9200'  // SSL to IP + Port
];
//扩展主机配置
/*客户端还支持 扩展的 主机配置语法。上述内联配置方式依赖于 PHP 中的 filter_var() 和 parse_url() 方法，使用它们完成 URL 组成部分的过滤验证和解析提取。很不幸的是，这些内置方法在执行某些特殊案例时会引发问题。例如， filter_var() 不能处理带有下划线的 URL（这样的 URL 是否合法取决于你对 RFC 是如何理解的）。类似地， parse_url() 在处理 password 中包含特殊字符（如 # 和 ? ）的 Basic Auth 时会出现阻塞。

基于上述原因，客户端需要支持一种能够在主机初始化时提供更高级控制能力的配置语法。以此消除如域名中带有下划线这样的特殊案例可能引发的问题。

扩展语法是为每个主机提供一个参数数组，如下所示*/
//推荐使用拓展主机配置
$hosts = [
    // 等于内联主机配置中使用 "https://username:password!#$?*abc@foo.com:9200/elastic"
    [
        'host' => 'foo.com',
        'port' => '9200',
        'scheme' => 'https',
        'path' => '/elastic',
        'user' => 'username',
        'pass' => 'password!#$?*abc'
    ],

    // 等于内联主机配置中使用 "http://localhost:9200/"
    [
        'host' => 'localhost',    
    ]
];
//对于每个配置的主机来说，只有 host 参数是必须的。 没有指定port时默认9200 没有指定scheme时默认http
$clientBuilder = ClientBuilder::create();
$clientBuilder->setHosts($hosts); 
$client = $clientBuilder->build(); 
//也可以使用连贯操作
$client = ClientBuilder::create()->setHosts($hosts)->build();
```

**授权和SSL**

详见[授权和SSL](auth.html)

**设置重试次数 set retries**

重试次数与连接池节点数关联，重试几次链接几个节点，当连接池中的这些节点都无法链接时，抛出OperationTimeoutException异常，并将失效的节点标记为无效节点

那些由于超过最大重试次数而抛出的异常都将包裹一个 MaxRetriesException 异常。你可以捕获一个特定的 curl 异常，然后通过调用 getPrevious() 方法检查其是否包裹了 MaxRetriesException 异常：

```php
$hosts = ['127.0.0.1:1'];

$client = ClientBuilder::create()->setHosts($hosts)->setRetries(0)->build();

try {
    $searchParams = [
        'index' => 'my_index',
        'body'  => [
            'query' => [
                'match' => [
                    'testField' => 'abc'
                ]
            ]
        ],
    ];
    $response = $client->search($searchParams);
    p($response);
}catch (Elasticsearch\Common\Exceptions\Curl\CouldNotConnectToHost $e) {
    
    $previous = $e->getPrevious();
    
    if( $previous instanceof Elasticsearch\Common\Exceptions\MaxRetriesException ) {
        echo 'Max retry';
    }
}
```

所有Curl的extension(CouldNotConnectToHost, CouldNotResolveHostException, 	OperationTimeoutException)都继承自TransportException，你可以先捕获 TransportException 然后检查在此之前的值：

```php
$client = ClientBuilder::create()->setHosts($hosts)->setRetries(0)->build();
try {
    $searchParams = [
        'index' => 'my_index',
        'body'  => [
            'query' => [
                'match' => [
                    'testField' => 'abc'
                ]
            ]
        ],
    ];
    $response = $client->search($searchParams);
    var_dump($response);
} catch (Elasticsearch\Common\Exceptions\TransportException $e) {
    
    $previous = $e->getPrevious();
    
    if ( $previous instanceof  Elasticsearch\Common\Exceptions\MaxRetriesException ) {
        echo 'Max retry';
    } 
    
}
```

**开启日志**

​	Elasticsearch-PHP 支持日志，但因性能原因默认未开启，如果需要开启，所有继承于PSR/Log的日志类都可以，推荐使用monolog, monolog另行安装，这里不做说明

```php
$logger = new Logger('mylog');
$logger->pushHandler(new StreamHandler('logpath/es.log', Logger::WARNING));

$client = ClientBuilder::create()->setHosts($hosts)->setRetries(0)->setLogger($logger)->build();
```

**设置http运行模式   configure http handler**

​	ES使用一个称为可交换的http传输层RingPHP，允许客户端构建通用的http请求，放到传输层执行。实际的执行信息对于客户端是隐藏的，也是模块化的。你可以根据需要从多个http处理程序进行选择。

​	ES使用的默认handler是组合型的handler， 当使用的是同步模式时， 使用CurlHandler，执行单个curl请求;
当使用的是异步（future）模式的时候，使用CurlMultiHandler， 基于curl multi接口实现，这需要更多性能开销，允许批量执行http请求。

​	推荐使用默认模式，允许快速同步执行的时候，同时保留了使用异步模式调用并行批处理的灵活性，如果不需要异步模式，则可以使用singleHandler来增加少许性能。

```php
//也可以自定义运行模式
class MyCustomHandler{
    
}

$defaultHandler = ClientBuilder::defaultHandler();
$singleHandler = ClientBuilder::singleHandler();
$multiHandler = ClientBuilder::multiHandler();
$customHandler = new MyCustomHandler();

$client = ClientBuilder::create()->setHandler($defaultHandler)->build();
```

**设置链接池**

```php
$connectionPool = '\Elasticsearch\ConnectionPool\StaticNoPingConnectionPool';
$client = ClientBuilder::create()
            ->setConnectionPool($connectionPool)
            ->build();
```

​	详见[链接池](connectPool.html)

**设置链接选择器**

```php
$selector = '\Elasticsearch\ConnectionPool\Selectors\StickyRoundRobinSelector';
$client = ClientBuilder::create()
            ->setSelector($selector)
            ->build();
```

详见[链接选择器](selector.html)

**设置序列化器**

ES 使用json序列化器 将php数组序列化成json 将json反序列化为数组，

```php
$serializer = '\Elasticsearch\Serializers\SmartSerializer';
$client = ClientBuilder::create()
            ->setSerializer($serializer)
            ->build();
```

​	详见[序列化器](serialize.html)

**设置自定义链接工厂**

当请求池发送请求时，ConnectionFactory 会实例化新的连接对象。一个连接对象代表一个节点。由于客户端将实际的网络工作交给 RingPHP，因此连接对象的主要工作是保持连接：节点是否是活节点？ Ping 的通吗？主机和端口是什么？

很少去自定义 ConnectionFactory ，如果你想那么做，你要提供一个完整的 ConnectionFactory 对象作为 `setConnectionFactory()` 方法的参数。这个对象应该实现 `ConnectionFactoryInterface` 接口。

```php
class MyConnectionFactory implements ConnectionFactoryInterface
{

    public function __construct($handler, array $connectionParams,
                                SerializerInterface $serializer,
                                LoggerInterface $logger,
                                LoggerInterface $tracer)
    {
       // Code here
    }


    /**
     * @param $hostDetails
     *
     * @return ConnectionInterface
     */
    public function create($hostDetails)
    {
        // Code here...must return a Connection object
    }
}


$connectionFactory = new MyConnectionFactory(
    $handler,
    $connectionParams,
    $serializer,
    $logger,
    $tracer
);

$client = ClientBuilder::create()
            ->setConnectionFactory($connectionFactory);
            ->build();
```

如上所述，如果你决定注入自定义 ConnectionFactory，请确保链接工厂类是正确的。ConnectionFactory 需要用到 HTTP handler，序列化器，日志和追踪。

**设置端点闭包 endpoint closure**

端点允许RESTful API或客户端操作ElasticSearch引擎中存储的数据，通过HTTP动词定义操作，通过URI定位数据资源，如查询端点GET /_search?q=user:kimchy、计数端点GET /_count?q=user:jim。

客户端使用 Endpoint 闭包发送 API 请求到正确的 Endpoint 对象上。一个命名空间对象会通过闭包构建一个新的 Endpoint ，这意味着如果你想拓展 API 的 endpoint，将会很方便的做到。

如下所示，我们新创建一个 endpoint：

```php
$transport = $this->transport;
$serializer = $this->serializer;

$newEndpoint = function ($class) use ($transport, $serializer) {
    if ($class == 'SuperSearch') {
        return new MyProject\SuperSearch($transport);
    } else {
        // Default handler
        $fullPath = '\\Elasticsearch\\Endpoints\\' . $class;
        if ($class === 'Bulk' || $class === 'Msearch' || $class === 'MPercolate') {
            return new $fullPath($transport, $serializer);
        } else {
            return new $fullPath($transport);
        }
    }
};

$client = ClientBuilder::create()
            ->setEndpoint($newEndpoint)
            ->build();
```

很明显，这样做你需要对现存的 endpoint 进行维护以保证其正常运行。同时你也需要确保端口和序列化都写入 endpoint 。

**使用hash配置初始化**

​	使用hash配置数组来实例化

```php
$configParams = [
    'hosts' => [
        'localhost:9200'
    ],
    'retries' => 2,
    'handler' => ClientBuilder::singleHandler()
];

$client = ClientBuilder::fromConfig($configParams);
```

如果传入不存在的配置项 会抛出一个exception 如果需要忽略不存在的配置项的话,将第二个参数设置为true

```php
$configParams = [
    'hosts' => [
        'localhost:9200'
    ],
    'retries' => 2,
    'handler' => ClientBuilder::singleHandler(),
    'notExistsConfig' => 5
];

ClientBuilder::fromConfig($configParams, true);
```



