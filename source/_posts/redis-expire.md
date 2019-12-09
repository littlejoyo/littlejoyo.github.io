---
title: Redis基础篇：初识过期键的设计
date: 2019-12-02 10:24:06
categories:
  - [数据库]
  - [Redis]
tag:
    - 数据库
    - Redis
    - 中间件
---
- Redis的数据是存储于内存之中，所以对内存的清理是非常重要的

- Redis提出了定期数据存储的设计，也就是数据在Redis上存储是有时间限制的

- 本篇主要从过期键的设计来对Redis进行了解

<!-- more -->
> Github issues:https://github.com/littlejoyo/Blog/issues/5

> 个人博客：https://littlejoyo.github.io/

> Redis是基于键值对存储形式的数据库，又因为数据存储的位置在内存，所以设置键值过期并及时清理是非常重要的。

# Redis如何设置键的过期时间？

- Redis中使用`EXPIRE `进行键时间的设置

- 使用`TTL`查询键剩余生存时间
```
redis> SET cache_page "www.google.com"
OK 
redis> EXPIRE cache_page 30  # 设置过期时间为 30 秒
(integer) 1
redis> TTL cache_page    # 查看剩余生存时间
(integer) 23
redis> EXPIRE cache_page 30000   # 更新过期时间
(integer) 1
redis> TTL cache_page
(integer) 29996
```

# Redis下的 Expire Key 

## Expire Key 底层的数据结构

Redis数据库在数据库服务器中使用了redisDb数据结构，结构如下：
```
typedef struct redisDb {
     dict *dict;     /* 键空间 key space */
     dict *expires;    /* 过期字典 */
     dict *blocking_keys;  /* Keys with clients waiting for data (BLPOP) */
     dict *ready_keys;   /* Blocked keys that received a PUSH */
     dict *watched_keys;   /* WATCHED keys for MULTI/EXEC CAS */
     struct evictionPoolEntry *eviction_pool; /* Eviction pool of keys */
     int id;      /* Database ID */
     long long avg_ttl;   /* Average TTL, just for stats */
} redisDb;
```

- 键空间(key space)：redisDb中的dict *dict成员就是将key和具体的对象（可能是string、list、set、zset、hash中任意类型之一）关联起来，存储着该数据库中所有的键值对数据。

- 过期字典(expires):保存数据库中所有键的过期时间，过期时间用UNIX时间戳表示，且值为long long整数

- id成员：id是数据库的编号，以整型表示。

## 设置过期时间的命令

- `EXPIRE \<key> \<ttl>`：命令用于将键key的过期时间设置为`ttl`秒之后
- `PEXPIRE \<key> \<ttl>`：命令用于将键key的过期时间设置为`ttl`毫秒之后
- `EXPIREAT \<key> \<timesramp>`：命令用于将key的过期时间设置为`timrestamp`所指定的秒数时间戳
- `PEXPIREAT \<key> \<timesramp>`：命令用于将key的过期时间设置为`timrestamp`所指定的毫秒数时间戳

## redisDb下expires的底层

- expires字段也是一个字典dict结构，字典的键为key，值为该key对应的过期时间

- 过期时间为long long类型整数，是以毫秒为单位的过期 UNIX 时间戳。

- 过期键的判定，其实是通过过期字典进行判定，步骤：

1.检查给定键是否存在于过期字典，如果存在，取出键的过期时间

2.通过判断当前UNIX时间戳是否大于键的过期时间，是的话，键已过期，相反则键未过期。

- 设置键值存储实例：
```
redis> set Ccww   5 2 0  
ok  
redis> expire Ccww 5  
ok  
```
使用redisDb结构存储数据图表示：([图片来源](https://juejin.im/post/5da7144ff265da5ba532b753#heading-12))

![image](https://i.loli.net/2019/12/09/d9Sk7n4NAwO6jMI.png)

# 过期键删除策略

## 三种删除策略
- **定时删除**：在设置键的过期时间的同时，创建一个定时任务，当键达到过期时间时，立即执行对键的删除操作

- **惰性删除**：放任键过期不管，但在每次从键空间获取键时，都检查取得的键是否过期，如果过期的话，就删除该键，如果没有过期，就返回该键

![惰性删除](https://i.loli.net/2019/12/09/bVHYqrMiQsEov31.png)


- **定期删除**：每隔一点时间，程序就对数据库进行一次检查，删除里面的过期键，至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。

## Redis对删除策略的选择

- Redis服务器结合**惰性删除**和**定期删除**两种策略一起使用。

- 通过这两种策略之间的配合使用，使得服务器可以在合理使用CPU时间和浪费内存空间取得平衡点。

# 过期键注意点

## 如果有大量的key需要设置同一时间过期，一般需要注意什么？

- 如果大量的key过期时间设置的过于集中，到过期的那个时间点，redis可能会出现短暂的卡顿现象。

- 严重的话会出现**缓存雪崩**，所以一般需要在时间上加一个随机值，使得过期时间分散一些，来避免缓存集中无效的现象。

- 造成缓存雪崩的场景：电商首页经常会使用定时任务刷新缓存，可能大量的数据失效时间都十分集中，如果失效时间一样，又刚好在失效的时间点大量用户涌入，就有可能造成缓存雪崩

## 什么是缓存雪崩？

- 缓存雪崩可以理解为缓存同一时间大面积失效，大量请求直接请求到数据库，最后数据库承担不了巨大请求服务掉线。

- 举个简单的例子：如果所有首页的Key失效时间都是12小时，中午12点刷新的，我零点有个秒杀活动大量用户涌入，假设当时每秒 6000 个请求，本来缓存在可以扛住每秒 5000 个请求，但是缓存当时所有的Key都失效了。

- 缓存失效过程，画图理解：
![缓存雪崩](https://i.loli.net/2019/12/09/4npbmDGMexfwtBv.png)

# RDB和AOF对过期键的处理

## RDB

### 生成RDB文件

- 程序会对数据库中的键进行检查，过期的键不会被保存到新创建的 RDB 文件中。

- 因此数据库中的过期键不会对生成新的 RDB 文件造成影响。

### 载入 RDB 文件

这里需要分情况说明：

 1. 如果服务器以**主服务器模式运行**，则在载入 RDB 文件时，程序会对文件中保存的键进行检查，过期键不会被载入到数据库中。所以过期键不会对载入 RDB 文件的主服务器造成影响。

 2. 如果服务器以**从服务器模式运行**，则在载入 RDB 文件时，不论键是否过期都会被载入到数据库中。但由于主从服务器在进行数据同步时，从服务器的数据会被清空。所以一般来说，过期键对载入 RDB 文件的从服务器也不会造成影响。

## AOF

### AOF 文件写入

- 当服务器以 AOF 持久化模式运行时，如果数据库某个过期键还没被删除，那么 AOF 文件不会因为这个过期键而产生任何影响，依旧保留。

- 而当过期键被删除后，那么程序会向 AOF 文件追加一条 DEL 命令来显式地记录该键被删除。

### AOF 重写

- 执行 AOF 重写过程中，也会被数据库的键进行检查，已过期的键不会被保存到重写后的 AOF 文件中。因此不会对 AOF 重写造成影响

# 主从复制对过期键的处理

- 当服务器运行在复制模式下，由主服务器来控制从服务器的删除过期键动作，目的是保证主从服务器数据的一致性。

- 那到底是怎么控制的呢？

1. **主服务器**删除一个过期键后，会向所有**从服务器**发送一个 DEL 命令，告诉**从服务器**删除这个过期键 

2. **从服务器**接受到命令后，删除过期键

PS:**从服务器**在接收到客户端对过期键的读命令时，依旧会返回该键对应的值给客户端，而不会将其删除。