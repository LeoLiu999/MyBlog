---
layout: post
title:  iptables 设置
date:   2019-12-03
categories: iptables
permalink : iptables/config.html
description: iptables 设置
show : true
---

##### iptables防火墙设置

先查看防火墙规则

```shell
iptables -L -n
```

清除原有的一些规则，使用自定义的

```shell
iptables -F   #清除预设表filter中的所有规则链的规则

iptables -X   #清除预设表filter中使用者自定链中的规则
```

开放22端口

```shell
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT 
```

开放80端口

```shell
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
#开放ping
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
```

接下来执行

```shell
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```

> **上面的意思是,当超出了IPTABLES里filter表里的两个链规则(INPUT,FORWARD)时,不在这两个规则里的数据包怎么处理呢,那就是DROP(放弃).应该说这样配置是很安全的.我们要控制流入数据包**
>
> **而对于OUTPUT链,也就是流出的包我们不用做太多限制,而是采取ACCEPT,也就是说,不在着个规则里的包怎么办呢,那就是通过.**
>
> **可以看出INPUT,FORWARD两个链采用的是允许什么包通过,而OUTPUT链采用的是不允许什么包通过.**
>
> **这样设置还是挺合理的,当然你也可以三个链都DROP,但这样做我认为是没有必要的,而且要写的规则就会增加.但如果你只想要有限的几个规则是,如只做WEB服务器.还是推荐三个链都是DROP.**

将配置的规则保存到iptables文件里，长期有效，然后重启iptables。

```shell
/usr/sbin/iptables-save  #centos7
service iptables restart #centos 7


iptables-save > /etc/iptables #ubuntu 将在命令行中配置的规则写入iptables文件
iptables-restore < /etc/iptables #ubuntu 从配置中加载规则
```

```shell
# 直接创建，目的是为了系统每次重启自动加载 ubuntu
root@cocosum:~# vi /etc/network/if-pre-up.d/iptables
 
# 内容
#!/bin/bash
iptables-restore < /etc/iptables
#给添加的脚本有执行的权限
chmod +x 
/etc/network/if-pre-up.d/iptables
#Ubuntu不能自动加载 不知道为什么

```

配置了规则之后发现无法连通外网，





