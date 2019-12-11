---
layout: post
title:  ElasticSearch-PHP安装
date:   2019-11-28
categories: ElasticSearch ElasticSearch-PHP install
permalink : ElasticSearch/install.html
description: Elasticsearch-PHP类库安装
---

##### 使用composer安装

```json
{
    "require": {
        "elasticsearch/elasticsearch": "~7.0"
    }
}
```

```shell
composer install
#composer update elasticsearch/elasticsearch
```

```php
require 'vendor/autoload.php';

$client = Elasticsearch\ClientBuilder::create()->build();
```

