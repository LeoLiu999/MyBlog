---
layout: post
title:  Redis基础数据结构
date:   2020-05-09
categories: redis
permalink : redis/datastruct.html
description: redis
show : true
---

redis是一种基于内存的key-value型数据库，特点：单线程worker（工作机制单线程），io操作多线程（ioThreads），支持高并发多链接数，链接池多，io并发大，多路复用器epoll。支持本地方法，io优化减少io，提高并发，计算向数据移动，操作为串行化、原子性，支持五种类型。

**memcache与redis的区别**

都是基于内存的key-value型数据库，但redis支持五种复杂数据类型且支持本地方法，memcache是数据向计算移动，而redis是计算向数据移动，io优化减少io，提高并发。

计算向数据移动：比如存数组进redis，取下标2的值时，可直接通过index取到下标为2的value，计算交给了redis内部。

**Redis的二进制安全**（富语言二进制安全）

将输入序列化为byte[]数组，统一接收byte[]数组，规避字节内存溢出。

##### Redis的五种数据类型

1. **string**  

   支持字符串操作、数值计算、二进制位计算

   + **字符串操作** 可以存字符串、二进制安全的东西、字节数组

   set 

   get

   append 字符串追加

   strlen 计算字符串长度(字节数)，与编码有关,如果value是数字，也是将数字当成string计算

   ​	**使用场景**

   1. 可以做session共享
   2. k-v缓存
   3. filesystem 基于内存级别的分布式fs文件系统，存储碎片小文件较佳，将文件名为key，文件二进制字节数组为value

   + **数值计算**

   incr 将数值增加value

   decr 将数值减少value

   **使用场景**

   1. 数值计算
   2. 计数器

   + **二进制位计算(bitmap)**

   bitmap 在计算机组成原理中学索引优化、内存压榨、数据传输、类型判定中出现的频率高

   setbit key offset value(0|1) 将二进制偏移量的位置设置成value  

   ----(符合ascii码编码规则,ascii码 0100 0000 这样的 以0开头,utf-8 ascii码扩展集  110x xxxx  10xx xxxx 前面两个1代表有几个字节，如果有三个字节，就多一个1，111x xxxx 10xx xxxx 10xx xxxx开头,前面几个1就代表需要几个字节  )

   bitcount key start end  从第几位开始到第几位结束里一共有几个二进制1	

   bitop operation destkey ...key 二进制计算 按位与、或、抑或计算=>bitop and resultkey key1 key2

   **使用场景** (做任意时间窗口统计、分析、索引、权限)

   1. 用户任意时间窗口的登录情况 setbit+bitcount

   2. 时间周期内活跃用户数 setbit+bitop+bitcount 将用户id转化为二进制位 二进制位多少取决于公司用户数，如果太大了，超过了512M，可以做分桶。

      setbit 20200510 用户id8 1 

      setbit 20200511  用户id3 1

      bitop or resultkey 20200510 20200511

      Bitcount resultkey

2. list 可满足栈与队列的行为

   lpush 从头部压入队列

   lpop 从头部取出队列

   rpush 从尾部压入队列

   rpop 从尾部取出队列

   lrange  key start stop获取队列数据

   ltrim key start stop 删除start到stop之外的数据 可用来优化list数据 保留多少个的list数据，用来保留活跃热数据

   **使用场景**

   1. 消息队列 消费者生产者、消息通知
   2. 热评论列表
   3. 栈
   4. 替代java容器 让jvm无状态、服务无状态

3. hash（hashtable、分治法）

   hset  设置hash

   hget 获取hash中某一个field

   hgetall 获取hash的全部field与value

   hkeys  获取hash的fields

   hvals 获取hash的values

   hincrby 将某一个field增加一定值 

   **使用场景**

   1. 存储聚集型数据
   2. 某些详情页（如商品详情页、用户详情）

4. set集合

   set特点：无序、去重

   集合操作不推荐使用，redis单线程的，集合的大量元素参与一个cup密集型操作，时间消耗很大，查起来很慢，在做集合操作的时候，后面的操作就被阻塞住了。优化方法：使用分布式多实例，一些服务器专门用来做集合的操作，让不同的网卡做不同的操作。

   sadd 往集合添加元素

   smembers 获取集合元素 不推荐使用，如果集合中元素太多，io就卡死了。

   srandmember key count 返回随机的一些元素  count可以为正或负，如果给的是正的，是去重的，如果给的数大于集合的元素个数，就返回去重的全部元素。如果给的是负的，就是可以重复的，如果给的数大于集合元素个数，也会返回该数个元素。

   spop 在集合中随机弹出一个元素

   sunion 获取集合的并集

   sinter 获取集合的交集

   sdiff key1 key2  获取key1针对对key2集合的差集 

   **使用场景**

   1. 随机事件 抽奖
   2. 验证码
   3. 扑克牌游戏
   4. 相同好友、相同买过某商品、相同玩过某游戏
   5. 好友推荐、商品推荐、游戏推荐

5. zset有序集合 sorted set

   zset特点：有序、去重

   zadd key score member 添加集合元素

   zrange key start end (withscore) 正序获取元素

   Zrevrange key start end (withscore) 倒序获取元素

   zrangeByScore 根据分值区间获取元素

   **使用场景**

   1. 排行榜
   2. 有序评论+分页

   **底层实现**

   动态排序实现方式->排序方式数据结构:

   元素少和小的时候 ziplist 压缩表

   元素多和大的时候 skiplist 跳跃表

   

   

   

