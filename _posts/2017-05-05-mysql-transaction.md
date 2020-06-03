```
layout: post
title:  mysql事务隔离级别
date:   2017-05-05
categories: mysql
permalink : mysql/transaction.html
description: mysql 事务隔离级别
show : true
```

##### mysql的事务隔离级别

1. **read uncommitted** 读未提交

   读未提交时的数据：当一个事务修改未提交时可读，此时读取到的数据是事务未提交时的数据，导致了脏读现象的出现，读取到的数据并不是准确的。

2. **read committed** 读提交

   读提交后的数据：当一个事务修改提交后另一个事务才可读，在此之前读取到的数据与事务提交后再读取到的数据不一致，虽然解决了脏读问题，但是出现了一个事务范围内两个相同的查询却返回了不同数据，称为不可重复读。

3. **Repeatable read** 重复读

   事务开启时（开始读取数据），不再允许其他事务的修改操作，必须等到当前事务完成才能进行下一个事务。

   但是这种情况还是会出现幻读问题，即允许insert操作，导致出现幻读现象。

4. **Serializable 序列化**

   Serializable 是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。

注:大多数数据库默认的事务隔离级别是Read committed，比如Sql Server , Oracle。Mysql的默认隔离级别是Repeatable read。





