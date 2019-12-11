---
layout: post
title:  ElasticSearch-ELKB系列之FileBeat
date:   2019-11-14
categories: ElasticSearch Beats FileBeat
permalink : ElasticSearch/FileBeat.html
description: ELKB之FileBeat
show : true
---

##### Beats简介

​	Beats是ES公司出品的一个数据收集工具，可用于收集各种服务器数据发送到Logstash或ElasticSearch。

**FileBeat**

​	Elastic提供了各种Beat供使用，这里我们选择FileBeat用于日志数据收集。

 1. 安装FileBeat

    deb

    ```shell
    curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.2-amd64.deb
    sudo dpkg -i filebeat-7.4.2-amd64.deb
    ```

    rpm

    ```shell
    curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.2-x86_64.rpm
    sudo rpm -vi filebeat-7.4.2-x86_64.rpm
    ```

    mac

    ```shell
    curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.2-darwin-x86_64.tar.gz
    tar xzvf filebeat-7.4.2-darwin-x86_64.tar.gz
    ```

    brew

    ```shell
    brew tap elastic/tap
    brew install elastic/tap/filebeat-full
    ```

 2. 配置FileBeat

    配置文件filebeat.yml

    * 抓取数据路径配置

      ```shell
      filebeat.inputs:
      - type: log
        enabled: true
        paths:
          - /var/log/*.log
      ```

    * 发送到ElasticSearch

      ```shell
      output.elasticsearch:
          hosts: ["192.168.1.42:9200"]
      ```

    * 如果你打算用Kibana仪表盘，可以这样配置Kibana

      ```shell
      setup.kibana:
            host: "localhost:5601"
      ```

    * 如果你的Elasticsearch和Kibana配置了安全策略,则需要指定访问凭证。

      ```shell
      output.elasticsearch:
            hosts: ["myEShost:9200"]
            username: "filebeat_internal"
            password: "{pwd}" 
      setup.kibana:
            host: "mykibanahost:5601"
            username: "my_kibana_user"  
            password: "{pwd}"
      ```

    * 使用Logstash

      ```shell
      output.logstash:
            hosts: ["127.0.0.1:5044"]
      ```

    * 加载索引模板

      ```shell
      setup.template.name: "your_template_name"
      setup.template.fields: "path/to/fields.yml"
      #覆盖一个已存在的模板
      setup.template.overwrite: true
      #禁用自动加载模板
      setup.template.enabled: false
      ```

    * 修改索引名称

      ```shell
      # 默认情况下，Filebeat写事件到名为filebeat-6.3.2-yyyy.MM.dd的索引，其中yyyy.MM.dd是事件被索引的日期。为了用一个不同的名字，你可以在Elasticsearch输出中设置index选项。例如：
      
      output.elasticsearch.index: "customname-%{[beat.version]}-%{+yyyy.MM.dd}"
      setup.template.name: "customname"
      setup.template.pattern: "customname-*"
      setup.dashboards.index: "customname-*"
      ```

      

 3. 启动FileBeat

    * 手动加载模板

      ```shell
      ./filebeat setup --template -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
      ```

    * 设置kibana dashboards

      ```shell
      ./filebeat setup --dashboards
      ```

    * 启动FileBeat

      ```shell
      ./filebeat -e -c filebeat.yml -d "publish"
      ```

    * 查看kibana仪表板示例

      ```shell
      http://127.0.0.1:5601
      ```

      