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

  vim filebeat.yml

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

logstash配置

创建my_pipeline.conf文件内容如下（该文件为pipeline配置文件，用于指定input，filter, output等）：

vim my_pipeline.conf

```yaml
 input {
        beats {
            port => "5044"
            codec = "json"
        }
    }
```

* Filter使用grok filter插件格式化数据，并使用geoip插件解析ip

  vim my_pipeline.conf

  ```js
  filter {
          grok {
              match => { "message" => "%{COMBINEDAPACHELOG}"}
          }
          geoip {
              source => "clientip"
          }
      }
  ```

* output 到kafka中

  ```js
  output{
  	kafka {
  		bootstrap_servers => "192.168.80.42:9092"
  		topic_id         => "test" 
    }
  }
  ```

* output到redis

  ```js
  output{
    redis {
  
          host => "192.168.1.100"   # redis主机地址
  
          port => 6379              # redis端口号
  
          password => "xxx"          # redis 密码
  
          #db => 2                   # redis数据库编号
  
          data_type => "channel"    # 使用发布/订阅模式
  
          key => "logstash_list_1"  # 发布通道名称
  
  	}
  }
  
  ```

* output到ES

  ```js
  output{
  
  	elasticsearch {
  
          hosts => "node18:9200"
  
          codec => json
  
          }
  
  }
  ```

**SSL加密传输**

logstash配置文件

vim ssl_pipeline.conf

```js
input {

    beats {

    port => 5044

    codec => "json"

    ssl => true
		//filebeat端传来的证书所在位置
   ssl_certificate_authorities => ["/usr/local/logstash-7.2.4/pki/tls/certs/filebeat.crt"]
	// 本端生成的证书所在的位置
   ssl_certificate => "/usr/local/logstash-7.2.4/pki/tls/certs/logstash.crt"
	//本端生成的密钥所在的位置
   ssl_key => "/usr/local/logstash-7.2.4/pki/tls/private/logstash.key"
   //需与ssl_certificate_authorities一起使用
		ssl_verify_mode => "force_peer"

       }

    syslog{

       }

}
```

Filebeat配置

vim filebeat.yml

```js
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
  output.logstash:
    hosts: ["127.0.0.1:5044"]
    ssl.certificate_authorities: ["/usr/local/filebeat-7.4.2/pki/tls/certs/logstash.crt"]
		ssl.certificate: "/usr/local/filebeat-7.4.2/pki/tls/certs/filebeat.crt"
  	ssl.key: "/usr/local/filebeat-7.4.2/pki/tls/private/filebeat.key" 
```



**将存入到redis、kafka里面的数据取出来发送到ES**

vim getdata.conf

```js
input{
	//从redis读取
 redis {

        host => "192.168.1.100"   # redis主机地址

        port => 6379              # redis端口号

        password => "xxx"          # redis 密码

        #db => 2                   # redis数据库编号

        data_type => "channel"    # 使用发布/订阅模式

        key => "logstash_list_1"  # 发布通道名称

	}
	//从kafka读取
	kafka {
		bootstrap_servers => "192.168.80.42:9092"
		topic_id         => "test" 
		auto_offset_reset => "earliest"
  }
	
  output {

    #输出到文件

    file {

        path => "/usr/local/logstash-5.6.10/data/log/logstash/all1.log" # 指定写入文件路径

#       message_format => "%{host} %{message}"         # 指定写入格式

        flush_interval => 0                             # 指定刷新间隔，0代表实时写入

     codec => json

       }

   #输出到es

   elasticsearch {

       hosts => "node18:9200"

       codec => json

       }
		}
  
  
  
}
```

**logstash同步mysql数据库数据到es（logstash5版本以上已集成jdbc插件，无需下载安装，直接使用）**

mysqlToEs.conf

```js
input {

 stdin { }

    jdbc {

        jdbc_connection_string => "jdbc:mysql://192.168.1.100:3306/leo-mysql"

        jdbc_user => "leo"

        jdbc_password => "123456"

   jdbc_driver_library => "/usr/local/logstash-7.2.4/mysql-connector-java-5.1.46.jar"

        jdbc_driver_class => "com.mysql.jdbc.Driver"

        jdbc_paging_enabled => "true"

        statement_filepath => "/usr/local/logstash-7.2.4/mysqlToEs.sql"

        #schedule => "* * * * *"

    }

 }
output {

     stdout {

        codec => json_lines

    }

    elasticsearch {

        hosts => "node18:9200"

        #index => "mainIndex"

        #document_type => "user"

        #document_id => "%{id}"

    }

}


```



mysqlToEs.sql

```sql
select * from sys_log
```



**Logstash启动**

```shell
bin/logstash -f my-pipeline.conf --config.reload.automatic #--config.reload.automatic选项的意思是启用自动配置加载，以至于每次你修改完配置文件以后无需停止然后重启Logstash
```

注：当修改了logstash配置文件的时候 无需重启logstash，但是如果input为filebeat，需要强制filebeat重头读取日志文件，为了这样做，你需要在终端停止filebeat，然后删除filebeat的注册文件，再重启filebeat。

```shell
rm filebeat/data/registr
```

**关于grok-filter插件**

grok-filter：可以将非结构化日志数据解析为结构化和可查询的内容

grok模式的语法是 %{SYNTAX:SEMANTIC}  SYNTAX是与文本匹配的模式的名称 SEMANTIC是匹配的文本提供的标识符

grok是通过系统预定义的正则表达式或者通过自己定义正则表达式来匹配日志中的各个值

正则解析式比较容易出错，建议先调试（地址）：

grok debugger调试：<http://grokdebug.herokuapp.com/>

grok事先已经预定义好了许多正则表达式规则，该规则文件存放路径：

Logstash/vendor/bundle/jruby/1.9/gems/logstash-patterns-core-4.1.2/patterns

1. 示例1

   ```js
   filter {
   
     grok {match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
   
     }
   
   }
   ```

   输入

   ```js
   55.3.244.1 GET /index.html 15824 0.043
   ```

   输出

   ```js
   client: 55.3.244.1（IP）
   
   method: GET（方法）
   
   request: /index.html（请求文件路径）
   
   bytes: 15824（字节数）
   
   duration: 0.043（访问时长）
   ```

2. 示例2

   ```js
   filter {
   
       grok {
   
           match => { "message" => "%{COMBINEDAPACHELOG}"}
   
       }
   
   }
   ```

   输入

   ```js
   192.168.80.183 - - [04/Jan/2018:05:13:42 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png
   
   HTTP/1.1" 200 203023 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel
   
   Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
   ```

   输出

   ```js
   "clientip" => "192.168.80.183",
   
   "timestamp" => "04/Jan/2018:05:13:42 +0000",
   
   "verb" => "GET",
   
   "request" => "/presentations/logstash-monitorama-2013/images/kibana-search.png",
   
   "referrer" => "\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"",
   
   "response" => "200",
   
   "bytes" => "203023",
   
   "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
   ```

3. 示例3  自定义grok表达式mypattern[A-Z]

   ```js
   filter {
   
     grok{
   　　match=>{
   　　　　"message"=>"%{IP:clientip}\s+(?<mypattern>[A-Z]+)"}
       }
   
   }
   ```

   输入

   ```js
   12.12.12.12 ABC
   ```

   输出

   ```js
   "clientip" => "12.12.12.12",
   "mypattern" => "ABC"
   ```

4. 示例4 移除重复字段

   ```js
   filter {
   
       grok {
   
           #match => { "message" => "%{COMBINEDAPACHELOG}"}
   
            match => { "message" => "%{IP:clientip}\s+%{IP:clientip1}"}
   
       }
   
       mutate {
   
       remove_field => ["message"]
   
       remove_field => ["host"]
   
      }
   
   }
   ```

5. 示例5

   ```js
   filter {
   
       grok {
   
            match => { "message" =>
   
    "%{DATA:ymd} %{DATA:sfm} %{DATA:http} %{DATA:info}  %{GREEDYDATA:index}"}
   
   }
   
   }
   ```

   【Data在pattern中的定义是：.\*? GREEDYDATA在pattern中的定义是：.\*】

   输入

   ```js
   2018-07-30 17:04:31.317 [http-bio-8080-exec-19] INFO  c.u.i.b.m.s.i.LogInterceptor - ViewName: modules/datashare/front/index
   ```

   输出

   ```js
   {
   
     "_index": "logstash-2018.07.31",
   
     "_type": "log",
   
     "_id": "AWTvhiPD6Wkp4mVEj3GU",
   
     "_version": 1,
   
     "_score": null,
   
     "_source": {
   
       "offset": 125,
   
       "input_type": "log",
   
       "index": "c.u.i.b.m.s.i.LogInterceptor - ViewName: modules/datashare/front/index",
   
       "source": "/home/usieip/bdp-datashare/logs/b.log",
   
       "type": "log",
   
       "tags": [],
   
       "ymd": "2018-07-30",
   
       "@timestamp": "2018-07-31T08:48:17.948Z",
   
       "@version": "1",
   
       "beat": {
   
         "name": "node183",
   
         "hostname": "node183",
   
         "version": "5.6.10"
   
       },
   
       "http": "[http-bio-8080-exec-19]",
   
       "sfm": "17:04:31.317",
   
       "info": "INFO"
   
     },
   
     "fields": {
   
       "ymd": [
   
         1532908800000
   
       ],
   
       "@timestamp": [
   
         1533026897948
   
       ]
   
     },
   
     "sort": [
   
       1533026897948
   
     ]
   
   }
   ```

**grok常用参数**

1. match：match作用：用来对字段的模式进行匹配

2. patterns_dir：用来指定规则的匹配路径，如果使用logstash自定义的规则时，不需要写此参数。Patterns_dir可以同时制定多个存放过滤规则的目录；

   ```js
   patterns_dir => ["/opt/logstash/patterns","/opt/logstash/extra_patterns"]
   ```

3. remove_field：如果匹配到某个”日志字段，则将匹配的这个日志字段从这条日志中删除（多个以逗号隔开)

   ```js
   remove_field => ["foo _％{somefield}"]
   ```

**其他filter**

1、 clone-filter：克隆过滤器用于复制事件

2、 drop-filter：丢弃所有活动

3、 json-filter：解析JSON事件

4、 kv-filter：解析键值对



**logstash配置文件**

- Pipeline配置文件，名称可以自定义，在启动Logstash时显式指定，编写方式可以参考前面示例，对于具体插件的配置方式参见具体插件的说明(使用Logstash时必须配置)： 用于定义一个pipeline，数据处理方式和输出源
- Settings配置文件(可以使用默认配置)： 在使用Logstash时可以不用设置，用于性能调优，日志记录等   
  - logstash.yml：用于控制logstash的执行过程   
  - pipelines.yml: 如果有多个pipeline时使用该配置来配置多pipeline执行
  - jvm.options：jvm的配置    
  - log4j2.properties:log4j 2的配置，用于记录logstash运行日志
  - startup.options: 仅适用于Lniux系统，用于设置系统启动项目！
- 为了保证敏感配置的安全性，logstash提供了配置加密功能

**扩展Logstash**

当单个Logstash无法满足性能需求时，可以采用横向扩展的方式来提高Logstash的处理能力。横向扩展的多个Logstash相互独立，采用相同的pipeline配置，另外可以在这多个Logstash前增加一个LoadBalance，以实现多个Logstash的负载均衡。

**Logstash性能调优**

1. **Inputs和Outputs的性能**：当输入输出源的性能已经达到上限，那么性能瓶颈不在Logstash，应优先对输入输出源的性能进行调优。

2. **系统性能指标**：

   - **CPU**：确定CPU使用率是否过高，如果CPU过高则先查看JVM堆空间使用率部分，确认是否为GC频繁导致，如果GC正常，则可以通过调节Logstash worker相关配置来解决。

   - **内存**：由于Logstash运行在JVM上，因此注意调整JVM堆空间上限，以便其有足够的运行空间。另外注意Logstash所在机器上是否有其他应用占用了大量内存，导致Logstash内存磁盘交换频繁。

   - **I/O使用率**：

     1) *磁盘IO*： 磁盘IO饱和可能是因为使用了会导致磁盘IO饱和的创建（如file output）,另外Logstash中出现错误产生大量错误日志时也会导致磁盘IO饱和。Linux下可以通过iostat, dstat等查看磁盘IO情况

     2) *网络IO*： 网络IO饱和一般发生在使用有大量网络操作的插件时。linux下可以使用dstat或iftop等查看网络IO情况

3. **JVM堆检查**：

   * 如果JVM堆大小设置过小会导致GC频繁，从而导致CPU使用率过高 
   * 快速验证这个问题的方法是double堆大小，看性能是否有提升。注意要给系统至少预留1GB的空间。
   * 为了精确查找问题可以使用jmap或VisualVM。
   * 设置Xms和Xmx为相同值，防止堆大小在运行时调整，这个过程非常消耗性能。

4. **Logstash worker设置**：

   worker相关配置在logstash.yml中，主要包括如下三个：

   * *pipeline.workers*：

     该参数用以指定Logstash中执行filter和output的线程数，当如果发现CPU使用率尚未达到上限，可以通过调整该参数，为Logstash提供更高的性能。建议将Worker数设置适当超过CPU核数可以减少IO等待时间对处理过程的影响。实际调优中可以先通过-w指定该参数，当确定好数值后再写入配置文件中。

   * *pipeline.batch.size*:

     该指标用于指定单个worker线程一次性执行flilter和output的event批量数。增大该值可以减少IO次数，提高处理速度，但是也以为这增加内存等资源的消耗。当与Elasticsearch联用时，该值可以用于指定Elasticsearch一次bluck操作的大小。

   * *pipeline.batch.delay*:

     该指标用于指定worker等待时间的超时时间，如果worker在该时间内没有等到pipeline.batch.size个事件，那么将直接开始执行filter和output而不再等待。





