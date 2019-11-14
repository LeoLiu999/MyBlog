---
layout: post
title:  ELKB系列之Logstash
date:   2019-11-14
categories: ElasticSearch Logstash
permalink : ElasticSearch/Logstash.html
description: ELKB之Logstash
---

**Logstash简介**

​	Logstash是ES公司出品的一款开源产品，是ELK系列中的L，主要用于收集、过滤和转储数据。

​	来自官网的介绍：

>**集中、转换和存储数据**
>
>Logstash 是开源的服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到您最喜欢的“存储库”中。

Logstash 主要由三部分组成，**inputs**(输入)->**filters**(过滤)->**outputs**(输出)，在inputs和outputs阶段可以使用**codecs**对数据格式进行处理，每个部分都在插件形式处理，用户可通过定义pipeline配置文件，设置需要使用的input、filter、codec、output插件。以实现特定的数据采集，数据处理，数据输出等功能。  

1. **inputs** 从数据源获取数据 常见的插件如file, syslog, redis, beats 等。
2. **filters** 用于处理数据  如格式转换、数据派生等，常见的插件如grok, mutate, drop,  clone, geoip等。
3. **outputs** 用于数据输出 常见的插件如elastcisearch，file, graphite, statsd等。
4. **codecs** Codecs不是一个单独的流程，而是在输入和输出等插件中用于数据转换的模块，用于对数据进行编码处理，常见的插件如json，multiline。

**Logstash 安装**

[官网下载地址](https://www.elastic.co/cn/downloads/logstash);

**Logstash执行模式：**

1. 每个Input启动一个线程，从对应数据源获取数据  
2. Input会将数据写入一个队列：默认为内存中的有界队列（意外停止会导致数据丢失）。为了防止数丢失Logstash提供了两个特性：
   * [Persistent Queues](https://www.elastic.co/guide/en/logstash/current/persistent-queues.html)：通过磁盘上的queue来防止数据丢失
   *  [Dead Letter Queues](https://www.elastic.co/guide/en/logstash/current/dead-letter-queues.html)：保存无法处理的event（仅支持Elasticsearch作为输出源）
3. Logstash会有多个pipeline worker, 每一个pipeline worker会从队列中取一批数据，然后执行filter和output（worker数目及每次处理的数据量均由配置确定）

**Logstash配置**

* inputs使用filebeat, filebeat配置：

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
  output.logstash:
    hosts: ["127.0.0.1:5044"]
```

​	其中的paths为需要抓取的日志文件目录。

启动filebeat命令：

```shell
./filebeat -e -c filebeat.yml -d "publish"
```

​	

​	filter







**Logstash启动**







