---
layout: post
title:  ElasticSearch-PHP未来模式(多线程模式)
date:   2019-11-28
categories: ElasticSearch ElasticSearch-PHP future
permalink : ElasticSearch/future.html
description: Elasticsearch-PHP未来模式
---

##### 未来模式（异步多线程模式）

未来模式允许客户端批量处理请求 (并行发送到群集)，对性能与吞吐量将会有巨大的提高。

PHP是单线程模式，但是libcurl提供了“多接口”的功能，该功能允许像php这样的语言通过提供一批要处理的请求来获得并发，批处理由底层多线程 libcurl 库并发执行，然后将响应返回给 PHP。

在单线程环境中，执行`n`个请求的时间，是`n`个请求延迟的时间总和。在多接口中，执行`n`个请求的时间是最慢请求的延迟（假如有足够多的句柄执行所有的请求）。

此外，多接口允许同时向不同主机发出请求，这意味着 Elasticsearch-PHP 客户端可以更有效的利用整个集群。

##### 使用未来模式

虽然使用这种模式相对简单，但它在代码中引入了更多的职责。为了开启 future 模式，在客户端选项中添加 `future` 参数，并将值设置为 `lazy` ：

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
```

这将会返回一个 *future* 对象，而不是一个真正的响应数据。future 对象是待处理对象，它看起来就像占位符。你可以把 future 对象当做普通对象在代码中使用。当你需要响应数据，你可以通过解析 future 对象获得。如果 future 对象已被解析（由于其它操作），可以立即使用响应数据。如果 future 对象没有被解析完成，那么解析动作将会产生阻塞，直到解析完成。

事实上，这意味着你可以通过设置 `future: lazy` 键值对来构造一个批量请求队列，而返回的 future 对象直到解析完成，程序才会继续执行。无论什么时候，所有的请求以并行的方式发送到集群，以异步的方式返回给 curl。

这听起来很复杂，但由于 RingPHP 的 `FutureArray` 接口，使其变得很简单，这使 future 对象看起来像一个简单的关联数组。例如：

```php
$client = ClientBuilder::create()->build();

$params = [
    'index' => 'test',
    'type' => 'test',
    'id' => 1,
    'client' => [
        'future' => 'lazy'
    ]
];

$future = $client->get($params);

$doc = $future['_source'];    // 此调用将产生阻塞并迫使 future 被解析
```

像普通返回一样，以关联数组的形式与 Future 模式交互，会导致 Future 模式解析出特定值（进而解析出所有待处理的请求和值）。可用模式如下：

```php
$client = ClientBuilder::create()->build();
$futures = [];

for ($i = 0; $i < 1000; $i++) {
    $params = [
        'index' => 'test',
        'type' => 'test',
        'id' => $i,
        'client' => [
            'future' => 'lazy'
        ]
    ];

    $futures[] = $client->get($params);     //请求入队列
}

foreach ($futures as $future) {
    // 访问Future模式的值，必要时会触发解析
    echo $future['_source'];
}
```

队列中的请求将会并行执行，并在执行之后给请求对应的 $future 变量赋值。默认每次批量执行 100 个请求。

如果你希望强制 Future 模式解析出返回值，但又不需要在当下使用，你可以调用 wait () 方法强制 Future 模式解析该请求：

```php
$client = ClientBuilder::create()->build();
$futures = [];

for ($i = 0; $i < 1000; $i++) {
    $params = [
        'index' => 'test',
        'type' => 'test',
        'id' => $i,
        'client' => [
            'future' => 'lazy'
        ]
    ];

    $futures[] = $client->get($params);     //请求入队列
}

//wait() 方法强制 Future 模式执行底层请求
$futures[999]->wait();
```

##### 修改批次大小

默认的批次大小是 100，也就是说每当有 100 个请求在排队，客户端就会开始处理 Future 对象（比如初始化一个 `curl_multi` 调用）。 批次的大小可以按照你的喜好来修改。 通过设置 Handler 的 `max_handles` 来修改。

```php
$handlerParams = [
    'max_handles' => 500
];

$defaultHandler = ClientBuilder::defaultHandler($handlerParams);

$client = ClientBuilder::create()
            ->setHandler($defaultHandler)
            ->build();
```

上面的设置会让客户端每 500 个请求进行一次批量处理。要注意的是，不论这个批次是否被填满，当开始处理一个 Future 对象时会导致整个批次被都处理。比如，只有 499 个请求在这个批次中... 但是最后一个 Future 对象的直接处理（也就是获取结果）会导致整个批次都被直接处理（可能会等待比较久，也可以用这个特性来触发批处理）：

```php
$handlerParams = [
    'max_handles' => 500
];

$defaultHandler = ClientBuilder::defaultHandler($handlerParams);

$client = ClientBuilder::create()
            ->setHandler($defaultHandler)
            ->build();

$futures = [];

for ($i = 0; $i < 499; $i++) {
    $params = [
        'index' => 'test',
        'type' => 'test',
        'id' => $i,
        'client' => [
            'future' => 'lazy'
        ]
    ];

    $futures[] = $client->get($params);     // 构建一个请求队列
}

// 处理了最后一个 Future 对象，前面的也都会被处理
$body = $future[499]['body'];
```

##### 多类型请求

一个批处理中可以包含各种类型的请求。比如，可以在一个批处理中加入 Get 请求、Index 请求和 Search 请求，这依然能被正常处理。

```php
$client = ClientBuilder::create()->build();
$futures = [];

$params = [
    'index' => 'test',
    'type' => 'test',
    'id' => 1,
    'client' => [
        'future' => 'lazy'
    ]
];

$futures['getRequest'] = $client->get($params);     // 第一个请求 Get

$params = [
    'index' => 'test',
    'type' => 'test',
    'id' => 2,
    'body' => [
        'field' => 'value'
    ],
    'client' => [
        'future' => 'lazy'
    ]
];

$futures['indexRequest'] = $client->index($params);       // 第二个请求 Index

$params = [
    'index' => 'test',
    'type' => 'test',
    'body' => [
        'query' => [
            'match' => [
                'field' => 'value'
            ]
        ]
    ],
    'client' => [
        'future' => 'lazy'
    ]
];

$futures['searchRequest'] = $client->search($params);      // 第三个请求 Search

// 处理 Future，也就是获取其中一个结果，整个批次都会被处理...这里会阻塞，直到批处理请求完成才会返回
$searchResults = $futures['searchRequest']['hits'];

// 这里会立即返回值，因为已经在前面批处理中完成了请求
$doc = $futures['getRequest']['_source'];
```

##### Future 模式注意事项

在使用 future 模式时，有几点需要注意。最重要也最明显的是：需要自己处理将要遇到的问题。这点通常比较细微，但有时会引入意想不到的并发症。

例如，如果你手动使用 `wait()`，你可能需要调用 `wait()` 多次，如果存在重试的话。这是因为每次重试都会引入另一层包裹的 futures，并且每个都需要被解决掉才能得到最终结果。

但是，如果通过 ArrayInterface 访问值，则不需要这样做 (例如： `$response['hits']['hits']`)，因为 FutureArrayInterface 将自动地并且完全地解析 future 提供的值。

另外一个需要注意的是，某些 APIs 将失去 「助手」功能。 例如： "exists" APIs (如： `$client→exists()`, `$client→indices()→exists`, `$client→indices→templateExists()` 等) 通常在正常操作下返回 true 或 false。

在运行 future 模式时，future 的延展取决于你的应用，这就意味着客户端无法再检查响应并返回简单的 true /false。相反地，你将看到 Elasticsearch 原始返回并须要采取适当的措施。

这也适用于 `ping()`。