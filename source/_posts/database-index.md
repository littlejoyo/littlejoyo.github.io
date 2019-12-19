---
title: 数据库系列:数据库索引什么情况下会失效呢？
date: 2019-12-17 14:48:44
categories:
  - [数据库]
  - [Mysql]
tag:
    - 数据库
    - Mysql
    - 索引
---

- 我们都知道，建立数据库索引可以大大提升数据的查询速度

- 但是你怎么就确定你的语句走了你设定的索引呢？

- 本篇重点介绍：数据库索引什么情况下会失效呢？查询的时候到底走没走索引！！

<!-- more -->
> Github issues:https://github.com/littlejoyo/Blog/issues/

> 个人博客：https://littlejoyo.github.io/

# 如何判断数据库查询操作走了索引呢？

## Explain 命令

- 使用sql命令`explain`可以显示 MySQL 如何使用索引来处理 SELECT 语句以及连接表。

## 使用方法

- 在select语句前加上 `EXPLAIN` 就可以，然后查看返回的信息进行判断。

```
EXPLAIN SELECT `name`,`age` FORM `a` WHERE `a`.`name`
```



[【EXPLAIN字段详解（传送门）】](https://littlejoyo.github.io/2019/12/17/database-explain/)



# 索引失效的各种情况和应对
## 1.不推荐使用select * 查询数据，可能会导致不走索引

- 如果你要查询的是`SELECT * FROM T WHERE Y=XXX;`

- 假如你的T表上有一个包含Y值的组合索引，但是优化器会认为需要一行行的扫描会更有效，这个时候，优化器可能会选择全表查询

- 但是如果换成了`SELECT Y FROM T WHERE Y = XXX`，优化器会直接去索引中找到Y的值，因为从B树中就可以找到相应的值。

> 注意：select * 查询数据，在某些情况下也是会走索引。

### 在Mysql InnoDB上不推荐使用select * 的原因：

- 1.`select *`杜绝了索引覆盖的可能性，而索引覆盖又是速度极快，效率极高，业界极为推荐的查询方式；

- 2.不需要的字段会增加数据传输的时间，即使mysql服务器和客户端是在同一台机器上，使用的协议还是tcp，通信也是需要额外的时间。

- 3.大字段，例如很长的`varchar`，`blob`，`text`。准确来说，长度超过728字节的时候，会把超出的数据放到另外一个地方，因此读取这条记录会增加一次io操作。

## 2. 索引列的值允许为NULL，导致COUNT(*)不能走索引

- 如果在B+树索引中有一个空值，那么查询诸如`SELECT COUNT(*) FROM T` 的时候，因为`HASHSET`中不能存储空值的，所以优化器不会走索引

- 有两种方式可以让索引有效，一种是修改为`SELECT COUNT(*) FROM T WHERE XXX IS NOT NULL`或者把这个列的属性改为`not null` (不能为空)

## 3.索引列上有函数运算，导致不走索引

- 如果在T表上有一个索引Y，但是你的查询语句是`SELECT * FROM T WHERE FUN(Y) = XXX`。这个时候索引也不会被用到，因为你要查询的列中所有的行都需要被计算一遍

- 如果要让这种sql语句的效率提高的话，需要在这个表上建立一个基于函数的索引，比如`CREATE INDEX IDX FUNT ON T(FUN(Y))`;

- 这种方式，数据库引擎会建立一个存储所有函数计算结果的值，再进行查询的时候就不需要进行计算了，因为很多函数存在不同返回值，因此必须标明这个函数是有固定返回值的。

## 4. 统计信息不准确，导致不走索引

- 很长时间没有做表分析，或者重新收集表状态信息了，导致了在数据字典中，表的统计信息是不准确的

- 这个情况下，可能会使用错误的索引，这个效率可能也是比较低的

## 5.谓词使用了不等于（<>, !=）

- 比如，`select id  from test where id<>100` 查询不走索引

- 应对：尝试将`<>`或者`!=`修改为`in`或者`or`取代，也可以尝试使用`> or <`进行优化。

## 6.like模糊查询'%li' 百分号在前

- `like`用来做模糊查询，用法主要是在匹配字段前后添加`%`

- 当`%`放在匹配字段前是不走索引的，放在后面才会走索引

## 7. 不符合最左前缀原则

- 建立联合索引后，查询的字段如果不符合最左前缀原则也不走索引

- 最左前缀原则限于篇幅，在此不展开

## 8.隐式转换导致不走索引

- 索引不适用于隐式转换的情况，比如`SELECT * FROM T WHERE Y = 1` 在Y上面有一个索引

- 但是Y列的类型是VARCHAR，那么Mysql会将上面的`1`进行一个隐式的转换，`SELECT * FROM T WHERE TO_NUMBER(Y) = 1`,这个时候也是有可能用不到索引的。

- 所以，写查询语句前需要明确好字段类型

## 9.表的数据库小或者需要选择大部分数据，不走索引

- 比如你的表只有几个数据块大小，而且可以被Mysql一次性抓取，那么就没有使用索引的必要了

- 因为抓取索引还需要去根据rowid从数据块中获取相应的元素值，因此在表特别小的情况下，索引没有用到是情理当中的事情。

## 10.order by (统计函数的索引问题)

- 1） 如果`select` 只查询索引字段，`order by` 索引字段会用到索引，要不然就是全表排列；

- 2） 如果有`where` 条件，比如`where high=1 order by high asc` 这样order by 也会用到索引！

- 这个结论不仅对order by有效，对其他需要排序的操作也有效。比如group by 、union 、distinct等。

### order by limit 问题
- `order by` 查询大量数据的时候在线上环境的sql中，切记必须使用`limit`

- 比如执行`select M,N from a where M = xxx order by id`会对全表的每条数据进行扫描，加上limit后就只扫描具体数目的数据。

#### order by id 不添加limit
![无limit](https://i.loli.net/2019/12/17/oh1KT3RmPNjbLIE.png)

#### order by id 添加limit
![有limit](https://i.loli.net/2019/12/17/KZa3pGfxR8j7thE.png)

> 注意：limit不影响order by 的索引选择，但是会影响扫描的rows数目，不添加limit限制的话，说明会花费更多的开销去扫描所有的查询结果。