---
title: "Novel Cloud"
date: 2022-11-04T19:14:32+08:00
math: true
slug: "Novel-Cloud"
# weight: 1
# aliases: ["/first"]
tags: ["项目","微服务"]
categories: ["项目"]
# author: ["Me", "You"] # multiple authors
# showToc: true
# TocOpen: false
# draft: false
# hidemeta: false
# comments: false
# description: "Desc Text."
image: "https://api.ixiaowai.cn/api/api.php"
---


# Novel Cloud

## 项目模块

### 门户首页

![](home.png)
首页包括 `小说推荐（包括轮播图、周推、强推等）`、`新闻公告`、`点击榜`、`新书榜`、`更新榜（包括最新更新列表）` 和 `友情链接` 6个内容区域的展示

`首页是我们小说门户的入口，承载着我们系统很大一部分流量，并且内容不需要实时更新。所以首页相关内容的查询最好都做缓存处理。`

### 新闻中心

### 用户中心

### 小说

### 支付

### 作家

### 文件

### 搜索

### 分布式锁

#### setnx命令（原子性操作）

获取锁（OK）->业务处理->释放锁
获取锁（NO）->等待重试

问题：如果业务处理异常或程序宕机，锁无法释放产生死锁
解决：设置锁的过期时间（expire）->获取锁并设置过期时间（set key value EX|PX time NX）

问题：如果业务处理时间超过锁的过期时间，锁无法释放产生死锁
解决：
1.值指定为uuid，防止删除别人的锁
2.删锁命令使用lua脚本
script

```Lua
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

value:uuid

```Java
redisTemplate.execute(new DefaultRedisScript<>(script, Long.class), Arrays.asList(key), value)
```

3.业务处理时间超过锁的过期时间，自动续期（watch key -> multi -> expire -> exec）

#### Redisson

配置方法（单节点， 集群使用setNodeAddress）
```Java
Config config = new Config();
// redis/ rediss
config.useSingleServer().setAddress("redis://ip:port");
RedissonClient redissonClient = Redisson.create(config);
```

##### 可重入锁（ReentrantLock）

```Java
RLock lock = redissonClient.getLock("lock");
lock.lock();
```

> 1.锁自动续期（默认每10s续期 TTL 30s）
> 2.加锁业务执行完成，锁自动释放

*Redisson解决死锁问题（看门狗）*

问题：如果业务执行时间超过锁的过期时间，锁不会自动续期，但是业务还没执行完，导致锁无法释放
解决：设置默认过期时间的1/3以上，看门狗自动续期

##### 公平锁（FairLock）

使用`getFairLock()`方法获取公平锁

##### 读写锁（ReadWriteLock）

```Java
RReadWriteLock lock = redissonClient.getReadWriteLock("lock");
lock.readLock().lock(); // 读锁
lock.writeLock().lock(); // 写锁
```

改数据加写锁，读数据加读锁
保证一定能读到最新数据，写锁是一个排他锁（互斥锁，独享锁），读锁是一个共享锁
> 写 + 读 ： 等待写锁释放
> 写 + 写 ： 阻塞方式
> 读 + 写 ： 等待读锁释放

##### 闭锁（CountDownLatch）

门卫等待所有人都走完才能关门

```Java
RCountDownLatch latch = redissonClient.getCountDownLatch("latch");
latch.trySetCount(5);
latch.await();
```

业务

```Java
RCountDownLatch latch = redissonClient.getCountDownLatch("latch");
latch.countDown();
```

> 闭锁在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

##### 信号量（Semaphore）

```Java
RSemaphore semaphore = redissonClient.getSemaphore("semaphore");
semaphore.release(); // 释放
semaphore.acquire(); // 获取
```

> 信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。

##### 缓存数据一致性问题

###### 双写模式（最终一致性）

数据库->更新->缓存

> 缓存暂时脏数据，缓存过期后自动更新

###### 失效模式（实时一致性）

写数据库->删除缓存->写缓存

![](cachedb.png)
