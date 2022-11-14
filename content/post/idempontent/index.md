---
title: "接口幂等性"
date: 2022-11-14T21:23:18+08:00
math: true
slug: "Idempontent"
# weight: 1
# aliases: ["/first"]
# tags: ["Tag1","Tag2"]
# categories: ["XCPC"]
# author: ["Me", "You"] # multiple authors
# showToc: true
# TocOpen: false
# draft: false
# hidemeta: false
# comments: false
# description: "Desc Text."
image: "https://api.ixiaowai.cn/api/api.php"
---

## 什么是幂等性

幂等性是指对于同一个请求，无论执行多少次，结果都是一样的。幂等性是指对于同一个请求，无论执行多少次，结果都是一样的。

## 解决方案

### 唯一索引

在数据库中，我们可以通过唯一索引来保证幂等性。例如，我们在数据库中创建一个唯一索引，当我们插入一条数据时，如果发现该数据已经存在，那么就不会插入，否则就插入。这样就保证了幂等性。

### 乐观锁

为了保证幂等性，我们可以使用乐观锁的原理实现。为数据字段增加一个version字段，每次更新数据时，version字段加1。当我们更新数据时，先查询出version字段，然后更新version字段加1，如果更新成功，那么就更新成功，否则就更新失败。

### 悲观锁

乐观锁可以实现的往往用悲观锁也能实现，在获取数据时进行加锁，当同时有多个重复请求时，只有一个请求能够获取到锁，其他请求就会被阻塞，直到获取到锁为止。这样就保证了幂等性。

### 分布式锁

#### 1.setnx命令（原子性操作）

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

#### 2.Redisson

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

#### Token机制

##### 什么是Token

Token是一种令牌，是服务端生成的一串字符串，客户端每次请求都必须带上这个令牌，服务端根据令牌来判断客户端的合法性

##### Token的作用

1. 服务端不用保存用户的登录状态，只需要保存Token即可
2. Token可以设置过期时间，过期后需要重新登录
3. Token可以设置权限，只有拥有权限的Token才能访问某些资源
4. Token可以设置刷新时间，过了刷新时间就需要重新登录

##### Token的实现

1.生成Token

```Java
public String createToken() {
    String token = UUID.randomUUID().toString().replace("-", "");
    return token;
}
```

2.存储Token

```Java
public void saveToken(String token, String username) {
    redisTemplate.opsForValue().set(token, username, 30, TimeUnit.MINUTES);
}
```

3.校验Token

```Java
public String checkToken(String token) {
    String username = redisTemplate.opsForValue().get(token);
    if (StringUtils.isEmpty(username)) {
        return null;
    }
    // 更新过期时间
    redisTemplate.expire(token, 30, TimeUnit.MINUTES);
    return username;
}
```

4.删除Token

```Java
public void deleteToken(String token) {
    redisTemplate.delete(token);
}
```
