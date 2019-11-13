---
layout: post
title:  Elastic Search原理解析
date:   2019-11-13
categories: ElasticSearch ES原理
permalink : ElasticSearch/BasicConcept.html
description: Elastic Search 原理解析
---

1. **倒排索引**

   倒排索引（反向索引），根据文章内容中的关键词建立索引。

   建立倒排索引后，一遍文章中索引量非常大，此时再使用正向索引，索引到文章标题，并建立索引矩阵，解决索引量爆炸的问题。

2. **搜索引擎原理**

   搜索引擎的核心即为建立倒排索引，只不过流程稍微复杂，包括了爬虫网页抓取、停顿词过滤（停顿词过滤：即一遍文章中没有意义的词，如的、而等这种没有实际意义不需要索引的词，即为所谓的分词。）等等。

   搜索引擎抓取到文章之后，对文章进行分词获取关键词，再根据关键词建立倒排索引。

   爬取内容-》分词-》建立倒排索引。	

3. **Elastic Search简介**

   在早期，业界有一个类库叫做Lucene，可以方便的建立倒排索引，但是Lucene本身是一个库，需要懂一些搜索引擎原理才能用的好，后来有人基于Lucene库进行封装，写出了Elastic Search（以下简称ES）。

   ES对搜索引擎的操作都封装成了restful的api，通过http请求进行操作。同时，ES还考虑了海量数据，实现了分布式，是一个可以存储海量数据的分布式搜索引擎。

4. **Elastic Search 基本概念**

   + 索引（index）

     ES的索引是存放数据的地方，可以理解为Mysql中的数据库

   + 类型（type）

     ES的类型可以理解为Mysql中的数据表，通过mapping定义数据结构，相当于表结构的描述，描述每个字段的类型。

   + Mapping

     mapping类似于数据库中的表结构定义，主要作用如下：

     1. 定义index下的字段名
     2. 定义字段类型，比如数值型、浮点型、布尔型等
     3. 定义倒排索引相关的设置，比如是否索引、记录position等

   + 文档（document）

     文档是最终的数据，一个文档即为一条记录。以JSON格式描述一行数据。

     举例：

     比如一首诗，有诗题、作者、朝代、字数、诗内容等字段，那么首先，我们可以建立一个名叫 Poems 的索引，然后创建一个名叫 Poem 的类型，类型是通过 Mapping 来定义每个字段的类型。

     比如诗题、作者、朝代都是 Keyword 类型，诗内容是 Text 类型，而字数是 Integer 类型，再把数据组织成 Json 格式存放进去。

     ```javascript
     //索引
     poems
     //类型
     'porm':{
       'properties':{
            'title':{
              'type':'keyword',
            },
            'author':{
              'type':'keyword', 
            },
            'dynasty':{
              'type':'keyword',
            },
            'words':{
              'type':'integer',
            }
            'content':{
              'type' : 'text'
            }  
       }
     }
     
     //文档
     {
       'title':'静夜思',
       'author':'李白',
       'dynasty':'唐',
       'words':20,
       'content':'床前明月光，疑是地上霜；举头望明月，低头思故乡。' 
     }
     ```

     keyword和text的的区别

     keyword类型不会分词，直接根据字符串内容建立反向索引，text类型先进行分词，再建立反向索引。

5. **Elastic Search分布式原理**

   * ES数据的分布式存储

     ​	ES会对数据进行切分，同时每一个分片会保存多个副本，来保证分布式环境下的高可用。

   * ES的master-slave架构

     ​	在ES中，节点是对等的，节点间会通过自己的一些规则选取集群的master，master会负责集群状态信息的改变，并同步给其他节点。

     ​	建立索引请求->master建立索引->同步到其他slave节点。

     ​	注意：只有建立索引和类型需要经过master，数据的写入有一个简单的routing规则，可以route到集群的任意节点，所以数据的写入压力是分散到整个集群的。

6. **ELKBK系统**

   ES的使用场景不仅是做搜索引擎，可以使用ES搭建ELK系统，用于做日志分析。

   E:Elastic Search  数据存储于搜索

   L:Logstash     数据格式化

   K:Kibana		数据可视化

   B:Beats          数据收集

   K:Kafka          消息队列，可用其他消息队列代替

   Beats收集日志数据，到Logstash作为生产者格式化之后写入Kafka等消息队列，再由Logstash作为消费者取出数据放入ES，最后通过Kibana平台进行数据的可视化。如果再接入一些实时计算模块，还能做实时报警功能。

   