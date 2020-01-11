---
title: Redis进阶篇：分布式锁的介绍和如何正确实现分布式锁
date: 2020-01-07 17:39:40
categories:
  - [数据库]
  - [Redis]
tag:
    - 数据库
    - Redis
    - 分布式锁
---

- 在单机部署的项目下，如果涉及到数据一致性的问题可以通过加锁的思路来解决

- 但是，越来越多的项目已经开始倾向于分布式系统，原单机并发控制锁策略已经失效

- 所以，需要提出一种能够实现跨域限制数据一致性的锁，也就是分布式锁的提出

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

> 个人博客：https://littlejoyo.github.io/

> 掘金：https://juejin.im/user/59c1c16f6fb9a00a4c270402

> 微信公众号：Joyo说

![weixin](https://i.loli.net/2020/01/11/NJIXozj5WAxgCiu.png)

# 1.为什么需要分布式锁？

- 分布式系统逐渐取代了传统单机系统项目，单纯使用开发语言的API加锁无法实现需求

- 在分布式系统下，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机并发控制锁策略失效

- 为了解决这个问题就需要一种跨域的互斥机制来控制共享资源的访问，这就是分布式锁的由来

# 2.什么是分布式锁？

> 要想理解分布式锁，需要和线程锁和进程锁进行对比来了解

## 2.1 线程锁

- 主要用来给方法、代码块加锁

- 当某个方法或代码使用锁，在同一时刻仅有一个线程执行该方法或该代码段

- Java开发环境下，线程锁只在同一JVM中有效果，因为线程锁的实现在根本上是依靠线程之间共享内存实现的，比如Synchronized、Lock等

## 2.2 进程锁

- 为了控制同一操作系统中多个进程访问某个共享资源

- 因为进程具有独立性，各个进程无法访问其他进程的资源

- 在Java开发环境下，无法通过`synchronized`等线程锁实现进程锁

## 2.3 分布式锁

- 当多个进程或者线程不在同一个系统中，用分布式锁控制多个进程和线程对资源的访问

# 3.分布式锁的特点

> 首先，为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：

- 1、互斥性：任意时刻，只能有一个客户端获取锁，不能同时有两个客户端获取到锁。

- 2、安全性：锁只能被持有该锁的客户端删除，不能由其它客户端删除。

- 3、死锁：获取锁的客户端因为某些原因（如宕机等）而未能释放锁，其它客户端再也无法获取到该锁。

- 4、容错：当部分节点（redis节点等）宕机时，客户端仍然能够获取锁和释放锁。

# 4.分布式锁的几种实现方式

1. 数据库乐观锁；

2. 基于ZooKeeper的分布式锁;

3. 基于Redis的分布式锁；

# 5.使用Redis实现分布式锁

## 5.1 加锁

### 5.1.1 代码实现

- 实现原理：基于Redis的命令`SET key value NX EX max-lock-time`

- java开发环境下，基于Redis实现分布式锁的加锁操作

```java
public class RedisTool {
private static final String LOCK_SUCCESS = “OK”;
private static final String SET_IF_NOT_EXIST = “NX”;
private static final String SET_WITH_EXPIRE_TIME = “PX”;
/**
* 尝试获取分布式锁
* @param jedis Redis客户端
* @param lockKey 锁
* @param requestId 请求标识
* @param expireTime 超期时间
* @return 是否获取成功
*/
public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {
String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
if (LOCK_SUCCESS.equals(result)) {
  return true;
  }
  return false;
  }
}
```

### 5.1.2 参数介绍

加锁就一行代码,这个set()方法一共有五个形参：`jedis.set(String key, String value, String nxxx, String expx, int time)`

- 第一个为`key`，我们使用key来当锁，因为key是唯一的。

- 第二个为`value`，我们传的是`requestId`，很多人这里可能不明白，有key作为锁不就够了吗，为什么还要用到value？原因就是我们在上面讲到`可靠性`时，分布式锁要满足第四个条件（安全性）解铃还须系铃人，通过给value赋值为requestId，我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。requestId可以使用UUID.randomUUID().toString()方法生成。

- 第三个为`nxxx`，这个参数我们填的是NX，意思是`SET IF NOT EXIST`，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；

- 第四个为`expx`，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。

- 第五个为`time`，与第四个参数相呼应，代表key的过期时间。 

### 5.1.3 执行结果

- **结果1**：当前没有锁（key不存在），那么就进行加锁操作，并对锁设置个有效期，同时value表示加锁的客户端。

- **结果2**：已有锁存在，不做任何操作。

### 5.1.4 分布式锁满足条件分析

- **互斥性**：首先，set()加入了NX参数，可以保证如果已有key存在，则函数不会调用成功，也就是只有一个客户端能持有锁

- **死锁**：其次，由于我们对锁设置了过期时间，即使锁的持有者后续发生崩溃而没有解锁，锁也会因为到了过期时间而自动解锁（即key被删除），不会发生死锁。

- **安全性**：最后，因为我们将value赋值为requestId，代表加锁的客户端请求标识，那么在客户端在解锁的时候就可以进行校验是否是同一个客户端。

- **容错性**：由于我们只考虑Redis单机部署的场景，所以容错性我们暂不考虑。

## 5.2 解锁

### 5.2.1 代码实现

```java
public class RedisTool {

    private static final Long RELEASE_SUCCESS = 1L;
    /** * 释放分布式锁 * @param jedis Redis客户端 * @param lockKey 锁 * @param requestId 请求标识 * @return 是否释放成功 */
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }
}
```

解锁只需要两行代码就搞定了！

- 第一行代码，我们写了一个简单的Lua脚本代码

- 第二行代码，我们将Lua代码传到jedis.eval()方法里，并使参数KEYS[1]赋值为lockKey，ARGV[1]赋值为requestId。

- eval()方法是将Lua代码交给Redis服务端执行。

> 那么这段Lua代码的功能是什么呢？

- 其实很简单，首先获取锁对应的value值，检查是否与requestId相等，如果相等则删除锁（解锁）。

- 那么为什么要使用Lua语言来实现呢？因为要确保上述操作是原子性的。

- 源于Redis的特性，执行eval()方法可以确保原子性


# 6.错误的加锁实例

> 基于Redis实现分布式锁的加锁和解锁操作，必须至少满足分布式锁的前三个条件，下面剖析一下错误的加锁和解锁操作。

## 6.1 加锁错误实例1

### 6.1.1 具体代码

比较常见的错误示例就是使用`jedis.setnx()`和`jedis.expire()`组合实现加锁，代码如下：

```java
public static void wrongGetLock1(Jedis jedis, String lockKey, String requestId, int expireTime) {

    Long result = jedis.setnx(lockKey, requestId);
    if (result == 1) {
        // 若在这里程序突然崩溃，则无法设置过期时间，将发生死锁
        jedis.expire(lockKey, expireTime);
    }
}
```

### 6.1.2 错误分析

- setnx()方法作用就是SET IF NOT EXIST，expire()方法就是给锁加一个过期时间。

- 由于这是两条Redis命令，不具有原子性，如果程序在执行完setnx()之后突然崩溃，导致锁没有设置过期时间。那么将会发生死锁。

- 原因：低版本的jedis不支持多参数的set()方法，导致操作违背了原子性（高并发场景要求，否则会出现数据不一致问题）

- 改正：用高版本直接使用set()多参数实现。

## 6.2 加锁错误实例2

### 6.2.1 具体代码

```java
public static boolean wrongGetLock2(Jedis jedis, String lockKey, int expireTime) {
    long expires = System.currentTimeMillis() + expireTime;
    String expiresStr = String.valueOf(expires);
    // 如果当前锁不存在，返回加锁成功
    if (jedis.setnx(lockKey, expiresStr) == 1) {
        return true;
    }
    // 如果锁存在，获取锁的过期时间
    String currentValueStr = jedis.get(lockKey);
    if (currentValueStr != null && Long.parseLong(currentValueStr) < System.currentTimeMillis()) {
        // 锁已过期，获取上一个锁的过期时间，并设置现在锁的过期时间
        String oldValueStr = jedis.getSet(lockKey, expiresStr);
        if (oldValueStr != null && oldValueStr.equals(currentValueStr)) {
            // 考虑多线程并发的情况，只有一个线程的设置值和当前值相同，它才有权利加锁
            return true;
        }
    }       
    // 其他情况，一律返回加锁失败
    return false;
}
```

1. 通过setnx()方法尝试加锁，如果当前锁不存在，返回加锁成功。

2. 如果锁已经存在则获取锁的过期时间，和当前时间比较，如果锁已经过期，则设置新的过期时间，返回加锁成功。

### 6.2.2 错误分析

 > 那么这段代码问题在哪里？
 
 1. 由于是客户端自己生成过期时间，所以需要强制要求分布式下每个客户端的时间必须同步。 
 
 2. 当锁过期的时候，如果多个客户端同时执行jedis.getSet()方法，那么虽然最终只有一个客户端可以加锁，但是这个客户端的锁的过期时间可能被其他客户端覆盖，不满足互斥性。
 
 3. 锁不具备拥有者标识，即任何客户端都可以解锁,不满足解铃人还需系铃人的原则。

# 7.错误的解锁实例

## 7.1 解锁错误实例1

### 7.1.1 代码

- 最常见的解锁代码就是直接使用jedis.del()方法删除锁

```
public static void wrongReleaseLock1(Jedis jedis, String lockKey) {
    jedis.del(lockKey);
}
```

### 7.1.2 错误分析

- 这种不先判断锁的拥有者而直接解锁的方式，会导致任何客户端都可以随时进行解锁，即使这把锁不是它的。

- 这种粗暴的解锁方式违背了互斥性和安全性，所以属于严重错误的解锁做法。

## 7.2 解锁错误实例2

### 7.2.1 具体代码

```java
public static void wrongReleaseLock2(Jedis jedis, String lockKey, String requestId) {     
    // 判断加锁与解锁是不是同一个客户端
    if (requestId.equals(jedis.get(lockKey))) {
        // 若在此时，这把锁突然不是这个客户端的，则会误解锁
        jedis.del(lockKey);
    }
}
```

### 7.2.2 错误分析

- 这种解锁代码乍一看也是没问题，与正确姿势差不多，唯一区别的是分成两条命令去执行，违背了操作的原子性

- 问题在于如果调用jedis.del()方法的时候，**这把锁已经不属于当前当前客户端**的时候会解除他人加的锁。

- 那么是否真的有这种场景？答案是肯定的，比如客户端A加锁，一段时间之后客户端A解锁，在执行jedis.del()之前，**锁突然过期了**，**此时客户端B尝试加锁成功**，然后客户端A再执行del()方法，则将客户端B的锁给解除了。

# 8.总结

- 本篇主要介绍了如何利用java代码正确实现Redi分布式锁的思路，代码只是提供思路，需要开发者自己去领悟思路，不要局限于java的实现方式。

- 通过正确实例和错误实例的对比，我们明白实现Redis分布式锁无非就是要满足分布式锁的四个条件（至少要满足除容错性的前三个条件）

- 如果项目中的Redis出现了多机部署的情况，那么就要考虑**容错性**（java开发场景下可以尝试使用Redisson实现分布式锁，这是Redis官方提供的Java组件。）

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)
