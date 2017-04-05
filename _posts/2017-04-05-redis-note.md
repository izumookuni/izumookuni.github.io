---
layout: post
title: Redis笔记
date: 2017-04-05 11:59:56 +08:00
author: "<Your Name Here>"
tags:
    - Redis
    - NoSql
---

**本文仅对Redis的操作进行一些简要介绍，暂时不写出本人不常使用的部分。**

## 安装

Redis下载地址：[https://redis.io/download](https://redis.io/download)

    make
    make install
    cd utils
    ./install_server

详细参考：[Redis 安装及配置](http://koda.iteye.com/blog/1257616)

可以在安装前使用`make test`来测试服务器性能，需要安装`tcl`包。

详细参考：[Redis Sentinel 高可用实现说明](http://www.cnblogs.com/zhoujinyi/p/5585723.html)

## 数据类型

Redis支持的数据类型：

+ 字符串（String）

+ 散列（Hash）

+ 列表（List）

+ 集合（Set）

+有序集合（Sorted Set）

### 字符串

常用命令：

    SET key value # 添加key
    GET key # 获取key
    DEL key # 删除key
    EXISTS key # 判断key是否存在
    KEYS <pattern> # 获得符合规则的key名列表
    TYPE key # 获得key值的数据类型

> Redis不支持嵌套，可以使用`user:1:friends`作为key名。

如果value是整型字符串：

    INCR key # 自增1，不存在的key值默认value为0
    DECR key # 自减1
    INCRBY key increment # 自增increment，适用于浮点数
    DECRBY key decrement # 自减decrement，适用于浮点数

其他命令：

    APPEND key value1 # 向键的value末尾追加value1，若key不存在，则相当于SET
    STRLEN key # 获得key值的字符串长度
    MSET key value [key value ...] # 同时添加多个key
    MGET key value [key value ...] # 同时获取多个key
    SETBIT key offset value # 位操作
    GETBIT key offset # 位操作

### 散列

常用命令：

    HSET key field value # 添加key
    HGET key field # 获取key指定field的value
    HDEL key field [field ...] # 删除key的指定field
    HMSET key field value [field value ...] # 同时添加key的多个field
    HMGET key field [field ...]  # 同时获得key的多个field的value
    HGETALL key # 获得指定key的所有field的value
    HEXISTS key field # 判断field是否存在
    HSETNX key field value # 当field不存在是赋值
    HINCRBY key field increment # 数值操作，以此类推
    
其他命令：

    HKEYS key # 只获取field名
    HVALS key # 只获取field值
    HLEN key # 获得field数量
    
### 列表

常用命令：‘

    LPUSH key value [value ...] # 向列表左端添加元素
    RPUSH key value [value ...] # 向列表右端添加元素
    LPOP key # 从列表左端弹出元素
    RPOP key # 从列表右端弹出元素
    LLEN key # 获取列表元素个数，key不存在时返回0
    LRANGE key start stop # 获得列表片段（包含两端），支持负数索引
    LREM key count value # 删除列表前count个值为value的元素，count＞0，从左边开始，count＜0，从右边开始，count＝0，删除所有值为value的元素
    
其他命令：

    LINDEX key index # 获得指定index的value
    LINDEX key index value # 设置index的value
    LINSERT key BEFORE | AFTER pivot value # 在列表中值为pivot的元素（从左向右查找）前面或后面插入value（能否根据index进行插入？）
    
### 集合

    SADD key value [value ...] # 增加元素
    SREM key value [value ...] # 删除元素
    SMEMBERS key # 获得集合中的所有元素
    SISMENBER key value # 判断元素是否在集合中
    SDIFF key1 [key2 ...] # 集合间运算 key1 - (key2 + ...) 
    SINTER key1 [key2 ...] # 集合间运算 key1 and key2 and ...
    SUNION key1 [key2 ...] # 集合间运算 key1 or key2 or ...
    
其他命令：

    SCARD key # 获得集合中元素个数
    SRANDMEMBER key [count] # 随机获得集合中count（默认为1）个元素，count＞0，元素不重复，count＜0，获得-count个元素，元素可能重复
    SPOP key # 随机弹出一个元素

### 有序集合

在集合类型的基础上有序集合类型为集合中的每个元素都关联了一个分数。

分数支持字数、双精度浮点数、正负无穷（+inf -inf）

常用命令：

    ZADD key score value [score value ...] # 增加元素
    ZSCORE key value # 获得元素的分数
    ZREM key value [value ...] # 删除元素
    ZRANGE key start stop [WITHSCORES] # 获得索引start到stop之间（包括两端）的所有元素（从小到大排序），添加WITHSCORES关键字以同时获得分数
    ZREVRANGE key start stop [WITHSCORES] # 同上（从大到小排序）
    ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offest count] # 获得指定分数范围范围的元素，在min、max前端添加`(`，结果将不包含端点；如有LIMIT关键字，则在获得的元素列表的基础上向后偏移offset个元素，并只获取前count个元素；ZREVRANGEBYSCORE同理
    ZINCRBY key increment value # 增加value的分数，允许负数
    
其他命令：
    ZCARD key # 获得集合中元素个数
    ZCOUNT key min max # 获得指定范围内的元素个数
    ZREMRANGEBYRANK key start stop # 按照排名范围（索引）删除元素
    ZREMRANGEBYSCORE key min max # 按照分数范围删除元素
    ZRANK key value # 获得元素的排名（索引），从小到大排列
    ZREVRANK key value # 同上，从大到小排列
    
## 进阶

### 事务（原子操作）

一组命令的集合，一个事务中的命令要么都执行，要么都不执行。

    MULTI
    <command1>
    <command2>
    ...
    EXEC
    
使用`WATCH`可以监控一个或多个键，保证键只能被修改一次，监控一直持续到EXEC命令结束。

    SET key 1
    WATCH key
    SET key 2
    MULTI
    SET key 3
    EXEC
    GET key
    
> 结果为2

### 过期时间

使用`EXPIRE`命令设置一个键的过期时间：`EXPIRE key time`，单位为秒。

使用`TTL`命令查看一个键还有多久会被删除：`TTL key`，单位为秒，`-2`表示已经删除，`-1`表示永久存在（默认情况）。

使用`PERSIST`命令取消键的过期时间设置：`PERSIST key`。

### 持久化

Redis支持RDB、AOF两种方式的持久化。

#### RDB

快照，当符合一定条件时Redis会自动将内存中的使用数据生成副本并存储在硬盘上。Redis会在以下几种情况下对数据进行快照：

1 根据配置规则进行自动快照

    save 900 1
    save 300 10
    save 60 10000
    
2 用户执行SAVE或BGSAVE命令

`SAVE`命令：快照执行的过程中会阻塞所有来自客户端的请求
`BGSAVE`命令：后台执行，不阻塞

3 执行FLUSHALL命令

会清除内存中的所有数据，并保存在磁盘中

#### AOF

需要开启AOF持久化，使得每次Redis每执行一次命令就写一次磁盘，只适合对数据读取速度不要求的场合。
