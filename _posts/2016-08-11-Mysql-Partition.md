---
layout: post
title:  Mysql：分区优化
date:   2016-08-11
categories: Mysql 分区
permalink : mysql/partition.html
description: mysql 分区优化
show : true
---

1. **分区类型**

  + range分区：使用某个字段做范围分区，范围区间要求连续且不能重叠，常用自增主键或时间列进行分区。

     + range columns分区：range分区的一种特殊类型。

       **不同之处：**

       1. range分区分区字段必须是整数型（或表达式结果为整数）,而range columns分区支持整数（tinyint到bigint）、字符串（char、varchar、binary、varbinary）、日期（date、datetime）。
    2. range分区接受表达式，而range columns不接受表达式，只能是列名。
       3. range columns分区允许多个列，是基于元组的比较，也就是基于字段组的比较，而range只能是一个列。
  
   + list分区：使用枚举型字段进行分区，该字段值有限且区分度不高。

   + hash分区：按给定的分区个数，将数据hash分配到不同的分区。

   + key分区：与hash分区相似，不同在于hash分区允许用户自定义表达式且只支持整数类型（或表达式返回整数）的字段，而key分区只能使用mysql提供的散列函数且支持使用除blob、text类型外的其他类型的列作为分区键。

     

2. 



