---
title: redis 锁的5个大坑，如何规避？
date: 2025-02-24 10:42:01
tags: Redis
categories: 架构设计
---


redis 锁的5个大坑，如何规避？
==================


分布式锁 使用场景和主要类型
--------------

### 分布式锁 的使用场景

Redis 分布式锁在多种场景下都非常有用，特别是在需要确保分布式系统中不同进程或线程对共享资源进行互斥访问的情况下。以下是一些常见的使用场景：

1.  **秒杀抢购或优惠券领取**：在电商平台中，当进行秒杀或领取优惠券时，需要确保同一时间只有一个用户能够成功操作，以避免超卖或重复领取的问题。

2.  **订单处理**：在分布式部署的电商系统中，用户下单前需要获取分布式锁，检查库存，确保库存足够后才允许下单，然后释放锁。

3.  **实时统计**：在需要统计在线用户数、PV、UV等实时数据时，可以使用分布式锁来避免并发冲突，确保数据的一致性。

4.  **任务调度**：在分布式系统中，如果需要执行任务调度，并且任务之间需要互斥执行，可以使用分布式锁来保证同一时间只有一个任务在运行。

5.  **分布式爬虫**：对于需要对同一网站进行抓取的分布式爬虫系统，可以使用分布式锁来避免多个爬虫同时抓取同一资源，导致IP被封或资源过载。

6.  **消息队列幂等性**：在使用 消息队列时，分布式锁可以用于确保消息不会被重复处理, 实现 幂等性。


#### 先看本地锁：

在单体的应用开发场景中，在多线程的环境下，涉及并发同步的时候，为了保证一个代码块在同一时间只能由一个线程访问， 一般可以使用synchronized语法和ReetrantLock去保证，这实际上是本地锁的方式。

也就是说，在同一个JVM内部，大家往往采用synchronized或者Lock的方式来解决多线程间的安全问题。但在分布式集群工作的开发场景中，在JVM之间，那么就需要一种更加高级的锁机制，来处理种跨JVM进程之间的线程安全问题.

#### 再看分布式锁

总之， 分布式场景， 可以使用分布式锁，它是控制分布式系统之间**互斥访问共享资源**的一种方式。

比如说在一个分布式系统中，多台机器上部署了多个服务，当客户端一个用户发起一个数据插入请求时，如果没有分布式锁机制保证，那么那多台机器上的多个服务可能进行并发插入操作，导致数据重复插入，对于某些不允许有多余数据的业务来说，这就会造成问题。而分布式锁机制就是为了解决类似这类问题，保证多个服务之间互斥的访问共享资源，如果一个服务抢占了分布式锁，其他服务没获取到锁，就不进行后续操作。

大致意思如下图所示（不一定准确）：

![](./2025/02/24/redis-锁的5个大坑，如何规避？/1.png)

### 什么是分布式锁？

**何为分布式锁？**

*   当在分布式模型下，数据只有一份（或有限制），此时需要利用锁的技术控制某一时刻修改数据的进程数。

*   用一个状态值表示锁，对锁的占用和释放通过状态值来标识。


**分布式锁的条件：**

*   互斥性。在任意时刻，只有一个客户端能持有锁。

*   不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。

*   具有容错性。只要大部分的 Redis 节点正常运行，客户端就可以加锁和解锁。

*   解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。


**分布式锁的实现：**

分布式锁的实现由很多种，文件锁、数据库、redis等等，比较多；分布式锁常见的多种实现方式：

1.  数据库悲观锁、

2.  数据库乐观锁；

3.  基于Redis的分布式锁；

4.  基于ZooKeeper的分布式锁。


在实践中，还是redis做分布式锁性能会高一些

### 常见分布式锁方案对比

| 分类 | 方案 | 实现原理 | 优点 | 缺点 |
| --- | --- | --- | --- | --- |
|基于数据库| 基于mysql 表唯一索引| 1.表增加唯一索引 2.加锁：执行insert语句，若报错，则表明加锁失败 3.解锁：执行delete语句| 完全利用DB现有能力，实现简单| 1.锁无超时自动失效机制，有死锁风险 2.不支持锁重入，不支持阻塞等待 3.操作数据库开销大，性能不高|
| 基于MongoDB findAndModify原子操作| 1.加锁：执行findAndModify原子命令查找document，若不存在则新增 2.解锁：删除document| 实现也很容易，较基于MySQL唯一索引的方案，性能要好很多|1.大部分公司数据库用MySQL，可能缺乏相应的MongoDB运维、开发人员 2.锁无超时自动失效机制|   
|基于分布式协调系统| 基于ZooKeeper| 1.加锁：在/lock目录下创建临时有序节点，判断创建的节点序号是否最小。若是，则表示获取到锁；否，则则watch /lock目录下序号比自身小的前一个节点 2.解锁：删除节点| 1.由zk保障系统高可用 2.Curator框架已原生支持系列分布式锁命令，使用简单| 需单独维护一套zk集群，维保成本高|
| 基于redis 命令的分布式锁| 基于redis命令| 1\. 加锁：执行setnx，若成功再执行expire添加过期时间 2. 解锁：执行delete命令| 实现简单，相比数据库和分布式系统的实现，该方案最轻，性能最好| 1.setnx和expire分2步执行，非原子操作；若setnx执行成功，但expire执行失败，就可能出现死锁 2.delete命令存在误删除非当前线程持有的锁的可能 3.不支持阻塞等待、不可重入|
| 基于redis Lua脚本能力（Redission分布式锁）| 1\. 加锁：执行SET lock\_name random\_value EX seconds NX 命令 2. 解锁：执行Lua脚本，释放锁时验证random\_value -- ARGV\[1\]为random\_value, KEYS\[1\]为lock\_nameif redis.call("get", KEYS\[1\]) == ARGV\[1\] then return redis.call("del",KEYS\[1\])else return 0end| 同上；实现逻辑上也更严谨，除了单点问题，生产环境采用用这种方案，问题也不大。| 不支持锁重入，不支持阻塞等待|   
 
实际上，大家业务场景中，用的还是 redis 锁。

在用redis 锁的过程中， 遇到过 的 5大深坑，大家看看， 你遇到 了吗？

redis 分布式锁 5大深坑
---------------

曾经，做O2O电商引流业务，要频繁的对商品库存进行扣减，为避免并发造成库存 超买超卖 等问题，采用 `redis` 分布式锁加以控制。

一路踩坑 ， 惨不忍睹 ， 经验深刻

之一： 原子性 之深坑
-----------

使用redis的分布式锁，最早的版本，用的是`setNx`命令。

`SETNX`（SET if Not eXists）命令用于设置一个键值对，但只有在键不存在的情况下。

因为它涉及到两个步骤：检查键是否存在和设置键。

如果多个进程或线程同时检查键是否存在，并且都发现键不存在，那么它们都可能会认为自己获得了锁。

以下是使用`SETNX`和Jedis实现一个非原子性Redis分布式锁的基本步骤：

1.  **尝试获取锁**： 使用`SETNX`命令尝试设置锁。如果返回值是1，表示获取锁成功；如果返回值是0，表示锁已经被其他进程或线程获取。

2.  **设置过期时间**： 为了确保锁最终会被释放，即使获取锁的进程或线程崩溃，也需要为锁设置一个过期时间。

3.  **执行业务逻辑**： 在获取锁之后，执行需要同步的业务逻辑。

4.  **释放锁**： 完成业务逻辑后，使用`DEL`命令删除锁。


以下是最早的版本，用的是`setNx` 分布式锁的示例代码：

```


import redis.clients.jedis.Jedis;  
  
public class RedisDistributedLock {  
    private static final String LOCK\_SUCCESS = "OK";  
    private Jedis jedis = null;  
    private String lockKey;  
    private String lockValue = UUID.randomUUID().toString();  
  
    public RedisDistributedLock(Jedis jedis, String lockKey) {  
        this.jedis = jedis;  
        this.lockKey = lockKey;  
    }  
  
    public boolean tryLock(int expireTime) {  
        // 尝试获取锁  
        String result = jedis.setnx(lockKey, lockValue);  
        if (LOCK\_SUCCESS.equals(result)) {  
            // 设置过期时间  
            jedis.expire(lockKey, expireTime);  
            return true;  
        }  
        return false;  
    }  
    /\*\*  
     \* 释放锁  
     \*/  
    public void unlock() {  
        // 检查锁是否由当前进程或线程持有  
        if (lockValue.equals(jedis.get(lockKey))) {  
            // 删除锁  
            jedis.del(lockKey);  
        }  
    }  
     
}  



```

**早期忽略了上面的原子性， 导致 有的场景， 加了锁没啥用。**

为什么呢？

*   上述代码中的 `lock`方法并不是原子性的，因为它涉及到两个步骤： 因为它涉及到`SETNX`和`EXPIRE`两个命令。

*   上述代码中的`unlock`方法并不是原子性的，因为它涉及到两个步骤：获取锁的值和删除锁。


如何解决 lock\`方法的原子性问题？答案是:

*   可以使用带`NX`和`EX`选项版本的 SET\`命令

*   或者使用lua脚本。


如何解决 unlock方法的原子性问题？答案是:

*   使用lua脚本。


其实 Redisson等分布式锁框架，就是使用 lua脚本实现。

不过，我们有的时候 不一定使用分布式锁框架，就像早期的版本一样， 喜欢自己造轮子。

下面，就是的第二个版本， 具备了 加锁和解锁 原子性的版本。

```


package com.crazymaker.springcloud.standard.lock;  
  
@Slf4j  
@Data  
@AllArgsConstructor  
public class JedisLock {  
  
    private  RedisTemplate redisTemplate;  
  
    private static final String LOCK\_SUCCESS = "OK";  
    private static final String SET\_IF\_NOT\_EXIST = "NX";  
    private static final String SET\_WITH\_EXPIRE\_TIME = "PX";  
  
    /\*\*  
     \* 尝试获取分布式锁  
     \* @param jedis Redis客户端  
     \* @param lockKey 锁  
     \* @param requestId 请求标识  
     \* @param expireTime 超期时间  
     \* @return 是否获取成功  
     \*/  
    public static   boolean tryLock(Jedis jedis, String lockKey, String requestId, int expireTime) {  
  
        String result = jedis.set(lockKey, requestId, SET\_IF\_NOT\_EXIST, SET\_WITH\_EXPIRE\_TIME, expireTime);  
  
        if (LOCK\_SUCCESS.equals(result)) {  
            return true;  
        }  
        return false;  
    }  
        
       public void unlock(Jedis jedis, String lockKey,  String requestId) {  
        // 释放锁的逻辑需要确保原子性，这里只是一个简单示例  
        // 实际上，这里应该使用Lua脚本来确保删除操作的原子性  
        String script =  
                "if redis.call('get', KEYS\[1\]) == ARGV\[1\] then " +  
                "return redis.call('del', KEYS\[1\]) " +  
                "else " +  
                "return 0 " +  
                "end";  
        jedis.eval(script, 1, lockKey, requestId);  
    }  
}  
  



```

**上面的加锁原子性，是通过使用带****`NX`和`EX`选项版本的 SET 命令解决。**

**上面的解锁原子性，是通过lua脚本解决。**

特别说说， `Jedis.set` 方法是 Jedis 客户端用来设置 Redis 中的键值对的方法，基本对应到 redis 的set命令， 版本有很多。

Jedis.set个方法有多个重载版本，以不同的方式设置键值对，例如设置键的过期时间等。

以下是 `Jedis.set` 方法的一些常见使用方式：

1.  `String set(String key, String value)`：将键值对设置到 Redis 中，如果键已经存在，则覆盖之前的值。返回 "OK" 表示操作成功。

2.  `String set(String key, String value, String nxxx)`：

    这个版本允许 指定一个条件，`nxxx` 可以是 `NX`（Not Exist）或 `XX`（eXist），分别表示只有当键不存在或已经存在时才设置值。

3.  `String set(String key, String value, String nxxx, String expx, int time)`：

    这个版本允许 设置键的过期时间，`expx` 可以是 `EX`（秒）或 `PX`（毫秒），`time` 是过期时间的值。


在源码层面，`Jedis.set` 方法的实现会将相应的命令发送到 Redis 服务器。例如，不带过期时间的 `set` 操作会发送 `SET` 命令，而带有过期时间的 `set` 操作会发送 `SET` 命令加上 `EX` 或 `PX` 选项。

上面最后的 `set`命令，是一个原子性的版本， 该命令可以指定多个参数。

```


String result = jedis.set(lockKey, requestId, "NX", "PX", expireTime);  
if ("OK".equals(result))   
{     
    return true;  
}  
return false;  



```

其中的参数介绍如下：

*   `lockKey`：锁的标识

*   `requestId`：请求id

*   `NX`：只在键不存在时，才对键进行设置操作。

*   `PX`：设置键的过期时间为 millisecond 毫秒。

*   `expireTime`：过期时间


这个 `set`命令是原子操作，加锁和设置超时时间，一个命令就能轻松搞定。

一看源码，就能发现，其实  `Jedis.set` 方法的不同参数，可以对应到 Redis 的SET命令的命令选项。

Redis 的 `SET` 命令用于将给定的键（key）与值（value）关联。Redis 的`SET` 命令的基本格式和命令选项如下：

```


SET key value  



```

此外，`SET` 命令还支持一些选项，以提供更多的控制功能，例如设置键的过期时间、仅在键不存在时设置值等。以下是 `SET` 命令的一些常用选项：

1.  **NX**：仅当键不存在时，才对键进行设置操作。相当于 "SET if Not eXists"。

    ```
    
    
    SET key value NX  
    
    
    
    ```

2.  **XX**：仅当键已经存在时，才对键进行设置操作。相当于 "SET if eXists"。

    ```
    
    
    SET key value XX  
    
    
    
    ```

3.  **EX**：设置键的过期时间，单位为秒。

    ```
    
    
    SET key value EX seconds  
    
    
    
    ```

4.  **PX**：设置键的过期时间，单位为毫秒。

    ```
    
    
    SET key value PX milliseconds  
    
    
    
    ```

5.  **EXAT**：设置键的过期时间，精确到某个时间点（Unix时间戳）。

    ```
    
    
    SET key value EXAT timestamp  
    
    
    
    ```

6.  **PXAT**：设置键的过期时间，精确到某个时间点（Unix时间戳），单位为毫秒。

    ```
    
    
    SET key value PXAT milliseconds-timestamp  
    
    
    
    ```

7.  **KEEPTTL**：设置键的值，但保持键的原始过期时间不变。

    ```
    
    
    SET key value KEEPTTL  
    
    
    
    ```

8.  **GET**：设置键的值，并返回旧的值。

    ```
    
    
    SET key value GET  
    
    
    
    ```


这些选项可以组合使用。

例如， 想设置一个键的值，并设置其过期时间为10秒，同时仅当键不存在时才进行设置，你可以使用如下命令：

```


SET key value NX EX 10  



```

`SET` 命令返回的结果通常是 "OK"，表示操作成功。如果使用了 `GET` 选项，`SET` 命令将返回旧的值。

在实际应用中，这些选项使得 `SET` 命令非常灵活，可以用于实现各种复杂的逻辑，如分布式锁、缓存控制等场景。

之二：连接耗尽 之深坑
-----------

早期的版本， 没有正确的被释放， 大致如下：

```


public class LockDemo {  
    public static void main(String\[\] args) {  
        JedisPool jedisPool = new JedisPool("localhost", 6379);  
        JedisLock lock = new JedisLock(jedisPool, "myLock");  
  
       if (lock.tryLock(10)) { // 尝试获取锁，锁的过期时间为10秒  
                
             // 执行业务逻辑  
              System.out.println("Lock acquired, executing business logic");  
                   
             lock.unlock(); // 执行完业务，释放锁  
             System.out.println("Lock released");  
               
         } else {  
              System.out.println("Failed to acquire lock");  
         }  
          
    }  
}  



```

上边代码中，如果业务代码异常，或者JVM在执行业务代码的时候重启，就会导致 未及时释放锁，导致其它线程会一直尝试获取锁阻塞，

导致连接耗尽：用`Jedis`客户端会报如下的错误信息

```


redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool  



```

redis线程池 已经没有空闲线程来处理客户端命令。

要解决这个问题，就需要及时的释放锁，并且过期的时间，不能设置太长。

改进的版本如下：

```


public class LockDemo {  
    public static void main(String\[\] args) {  
        JedisPool jedisPool = new JedisPool("localhost", 6379);  
        JedisLock lock = new JedisLock(jedisPool, "myLock");  
  
        try {  
            if (lock.tryLock(10)) { // 尝试获取锁，锁的过期时间为10秒  
                try {  
                    // 执行业务逻辑  
                    System.out.println("Lock acquired, executing business logic");  
                    Thread.sleep(5000); // 模拟业务逻辑执行时间  
                } finally {  
                    lock.unlock(); // 释放锁  
                    System.out.println("Lock released");  
                }  
            } else {  
                System.out.println("Failed to acquire lock");  
            }  
        } finally {  
            jedisPool.close();  
        }  
    }  
}  



```

### 注意事项

*   **锁的过期时间**：在 `tryLock` 方法中设置了锁的过期时间，这是为了防止持有锁的进程崩溃，从而导致锁永远无法释放。但是也不能太长，设置在 30s内为适宜。

*   **锁的安全性**：在 `unlock` 方法中，我们使用 Lua 脚本来确保检查和删除操作的原子性，这是为了防止错误地释放其他进程持有的锁。

*   **连接池资源管理**：在示例中，关闭了连接，确保 Jedis 连接在使用后被正确关闭，避免资源泄露。


所以，加锁后，如果不及时释放锁，会有很多问题。

分布式锁更合理的用法是：

1.  手动加锁

2.  业务操作

3.  手动释放锁

4.  如果手动释放锁失败了，则达到超时时间，redis会自动释放锁。


大致流程图如下：

![](./2025/02/24/redis-锁的5个大坑，如何规避？/2.png)

之三：锁过期 的深坑
----------

下面有一个简单的使用锁， 实现并发操作文件的例子，锁的时间 10秒：

```


  //写数据到文件  
function writeData(filename, data) {  
    boolean locked = lock.tryLock(10, TimeUnit.SECONDS);  
    if (!locked) {  
        throw 'Failed to acquire lock';  
    }  
  
    try {  
        //将数据写到文件  
        var file = storage.readFile(filename);  
        var updated = updateContents(file, data);  
        storage.writeFile(filename, updated);  
    } finally {  
        lock.unlock();  
    }  
}


```

问题是：

*   如果在写文件过程中，发生了 fullGC，并且其时间跨度较长， 超过了10秒，

*   那么，由于锁的有效期就是 10s，这时候任务没有执行完成，分布式锁就自动过期了。


在此过程中，client2 抢到锁，写了文件。

回到 client1： client1 的fullGC完成后，也继续写文件，**注意，此时client1 的并没有占用锁**，此时写入会导致文件数据错乱，发生线程安全问题。

fullGC STW导致，导致了的锁过期问题。 整个流程，打造具体如下图所示：

![](./2025/02/24/redis-锁的5个大坑，如何规避？/3.png)

### 锁过期问题 的解决方案

锁过期问题,大概的解决方案 有2种：

1： 模拟CAS乐观锁的方式，增加版本号

2：**watch dog自动延期机制**

### 方式一：模拟CAS乐观锁的方式，增加版本号

**1： 模拟CAS乐观锁的方式，增加版本号（如下图中的token）**

CAS乐观锁 的方法也很简单：

在每次写操作时加入一个 token。 token 可以是一个递增的数字（lock service 可以做到），每次有 client 申请锁就递增一次。比如：

*   client1 的token 是33

*   client2 的token 是34


紧接着 client1 活过来之后尝试写入数据，自身 token 33 比 34 小，因此client1 的写入操作被拒绝了。

![](./2025/02/24/redis-锁的5个大坑，如何规避？/4.png)

此方案如果要实现，需要调整业务逻辑与之配合，所以会入侵代码。

### 方式二：watch dog自动延期机制

客户端1加锁的锁key默认生存时间才30秒，如果超过了30秒，客户端1还想一直持有这把锁，怎么办呢？

简单！

只要客户端1一旦加锁成功，就会启动一个watch dog看门狗，**他是一个后台线程，会每隔10秒检查一下**，如果客户端1还持有锁key，那么就会不断的延长锁key的生存时间。

> redission，采用的就是这种方案， 此方案不会入侵业务代码。

watch dog看门狗 的作用是在锁没有过期之前，不断的延长锁的有效期。

默认情况下，锁的过期时间是 30 秒，看门狗的续期时间是 10 秒，

也可以通过修改 Config.lockWatchdogTimeout 来指定。


Redission 就是 使用 看门狗的机制。建议大家不用自己实现，直接使用Redission。

**当然，作为高级开发或者架构师， 锁过期的 原理还是要懂的。**

`redisson`的解决锁过期的源码，大致如下。

```


@Slf4j  
@Service  
public class RedisDistributionLockPlus {  
   
    /\*\*  
     \* 加锁超时时间，单位毫秒， 即：加锁时间内执行完操作，如果未完成会有并发现象  
     \*/  
    private static final long DEFAULT\_LOCK\_TIMEOUT = 30;  
   
    private static final long TIME\_SECONDS\_FIVE = 5 ;  
   
    /\*\*  
     \* 每个key的过期时间 {@link LockContent}  
     \*/  
    private Map<String, LockContent> lockContentMap = new ConcurrentHashMap<>(512);  
   
    /\*\*  
     \* redis执行成功的返回  
     \*/  
    private static final Long EXEC\_SUCCESS = 1L;  
   
    /\*\*  
     \* 获取锁lua脚本， k1：获锁key, k2：续约耗时key, arg1:requestId，arg2：超时时间  
     \*/  
    private static final String LOCK\_SCRIPT = "if redis.call('exists', KEYS\[2\]) == 1 then ARGV\[2\] = math.floor(redis.call('get', KEYS\[2\]) + 10) end " +  
            "if redis.call('exists', KEYS\[1\]) == 0 then " +  
               "local t = redis.call('set', KEYS\[1\], ARGV\[1\], 'EX', ARGV\[2\]) " +  
               "for k, v in pairs(t) do " +  
                 "if v == 'OK' then return tonumber(ARGV\[2\]) end " +  
               "end " +  
            "return 0 end";  
   
    /\*\*  
     \* 释放锁lua脚本, k1：获锁key, k2：续约耗时key, arg1:requestId，arg2：业务耗时 arg3: 业务开始设置的timeout  
     \*/  
    private static final String UNLOCK\_SCRIPT = "if redis.call('get', KEYS\[1\]) == ARGV\[1\] then " +  
            "local ctime = tonumber(ARGV\[2\]) " +  
            "local biz\_timeout = tonumber(ARGV\[3\]) " +  
            "if ctime > 0 then  " +  
               "if redis.call('exists', KEYS\[2\]) == 1 then " +  
                   "local avg\_time = redis.call('get', KEYS\[2\]) " +  
                   "avg\_time = (tonumber(avg\_time) \* 8 + ctime \* 2)/10 " +  
                   "if avg\_time >= biz\_timeout - 5 then redis.call('set', KEYS\[2\], avg\_time, 'EX', 24\*60\*60) " +  
                   "else redis.call('del', KEYS\[2\]) end " +  
               "elseif ctime > biz\_timeout -5 then redis.call('set', KEYS\[2\], ARGV\[2\], 'EX', 24\*60\*60) end " +  
            "end " +  
            "return redis.call('del', KEYS\[1\]) " +  
            "else return 0 end";  
    /\*\*  
     \* 续约lua脚本  
     \*/  
    private static final String RENEW\_SCRIPT = "if redis.call('get', KEYS\[1\]) == ARGV\[1\] then return redis.call('expire', KEYS\[1\], ARGV\[2\]) else return 0 end";  
   
   
    private final StringRedisTemplate redisTemplate;  
   
    public RedisDistributionLockPlus(StringRedisTemplate redisTemplate) {  
        this.redisTemplate = redisTemplate;  
        ScheduleTask task = new ScheduleTask(this, lockContentMap);  
        // 启动定时任务  
        ScheduleExecutor.schedule(task, 1, 1, TimeUnit.SECONDS);  
    }  
   
    /\*\*  
     \* 加锁  
     \* 取到锁加锁，取不到锁一直等待知道获得锁  
     \*  
     \* @param lockKey  
     \* @param requestId 全局唯一  
     \* @param expire   锁过期时间, 单位秒  
     \* @return  
     \*/  
    public boolean lock(String lockKey, String requestId, long expire) {  
        log.info("开始执行加锁, lockKey ={}, requestId={}", lockKey, requestId);  
        for (; ; ) {  
            // 判断是否已经有线程持有锁，减少redis的压力  
            LockContent lockContentOld = lockContentMap.get(lockKey);  
            boolean unLocked = null == lockContentOld;  
            // 如果没有被锁，就获取锁  
            if (unLocked) {  
                long startTime = System.currentTimeMillis();  
                // 计算超时时间  
                long bizExpire = expire == 0L ? DEFAULT\_LOCK\_TIMEOUT : expire;  
                String lockKeyRenew = lockKey + "\_renew";  
   
                RedisScript<Long> script = RedisScript.of(LOCK\_SCRIPT, Long.class);  
                List<String> keys = new ArrayList<>();  
                keys.add(lockKey);  
                keys.add(lockKeyRenew);  
                Long lockExpire = redisTemplate.execute(script, keys, requestId, Long.toString(bizExpire));  
                if (null != lockExpire && lockExpire > 0) {  
                    // 将锁放入map  
                    LockContent lockContent = new LockContent();  
                    lockContent.setStartTime(startTime);  
                    lockContent.setLockExpire(lockExpire);  
                    lockContent.setExpireTime(startTime + lockExpire \* 1000);  
                    lockContent.setRequestId(requestId);  
                    lockContent.setThread(Thread.currentThread());  
                    lockContent.setBizExpire(bizExpire);  
                    lockContent.setLockCount(1);  
                    lockContentMap.put(lockKey, lockContent);  
                    log.info("加锁成功, lockKey ={}, requestId={}", lockKey, requestId);  
                    return true;  
                }  
            }  
            // 重复获取锁，在线程池中由于线程复用，线程相等并不能确定是该线程的锁  
            if (Thread.currentThread() == lockContentOld.getThread()  
                      && requestId.equals(lockContentOld.getRequestId())){  
                // 计数 +1  
                lockContentOld.setLockCount(lockContentOld.getLockCount()+1);  
                return true;  
            }  
   
            // 如果被锁或获取锁失败，则等待100毫秒  
            try {  
                TimeUnit.MILLISECONDS.sleep(100);  
            } catch (InterruptedException e) {  
                // 这里用lombok 有问题  
                log.error("获取redis 锁失败, lockKey ={}, requestId={}", lockKey, requestId, e);  
                return false;  
            }  
        }  
    }  
   
   
    /\*\*  
     \* 解锁  
     \*  
     \* @param lockKey  
     \* @param lockValue  
     \*/  
    public boolean unlock(String lockKey, String lockValue) {  
        String lockKeyRenew = lockKey + "\_renew";  
        LockContent lockContent = lockContentMap.get(lockKey);  
   
        long consumeTime;  
        if (null == lockContent) {  
            consumeTime = 0L;  
        } else if (lockValue.equals(lockContent.getRequestId())) {  
            int lockCount = lockContent.getLockCount();  
            // 每次释放锁， 计数 -1，减到0时删除redis上的key  
            if (--lockCount > 0) {  
                lockContent.setLockCount(lockCount);  
                return false;  
            }  
            consumeTime = (System.currentTimeMillis() - lockContent.getStartTime()) / 1000;  
        } else {  
            log.info("释放锁失败，不是自己的锁。");  
            return false;  
        }  
   
        // 删除已完成key，先删除本地缓存，减少redis压力, 分布式锁，只有一个，所以这里不加锁  
        lockContentMap.remove(lockKey);  
   
        RedisScript<Long> script = RedisScript.of(UNLOCK\_SCRIPT, Long.class);  
        List<String> keys = new ArrayList<>();  
        keys.add(lockKey);  
        keys.add(lockKeyRenew);  
   
        Long result = redisTemplate.execute(script, keys, lockValue, Long.toString(consumeTime),  
                Long.toString(lockContent.getBizExpire()));  
        return EXEC\_SUCCESS.equals(result);  
   
    }  
   
    /\*\*  
     \* 续约  
     \*  
     \* @param lockKey  
     \* @param lockContent  
     \* @return true:续约成功，false:续约失败（1、续约期间执行完成，锁被释放 2、不是自己的锁，3、续约期间锁过期了（未解决））  
     \*/  
    public boolean renew(String lockKey, LockContent lockContent) {  
   
        // 检测执行业务线程的状态  
        Thread.State state = lockContent.getThread().getState();  
        if (Thread.State.TERMINATED == state) {  
            log.info("执行业务的线程已终止,不再续约 lockKey ={}, lockContent={}", lockKey, lockContent);  
            return false;  
        }  
   
        String requestId = lockContent.getRequestId();  
        long timeOut = (lockContent.getExpireTime() - lockContent.getStartTime()) / 1000;  
   
        RedisScript<Long> script = RedisScript.of(RENEW\_SCRIPT, Long.class);  
        List<String> keys = new ArrayList<>();  
        keys.add(lockKey);  
   
        Long result = redisTemplate.execute(script, keys, requestId, Long.toString(timeOut));  
        log.info("续约结果，True成功，False失败 lockKey ={}, result={}", lockKey, EXEC\_SUCCESS.equals(result));  
        return EXEC\_SUCCESS.equals(result);  
    }  
   
   
    static class ScheduleExecutor {  
   
        public static void schedule(ScheduleTask task, long initialDelay, long period, TimeUnit unit) {  
            long delay = unit.toMillis(initialDelay);  
            long period\_ = unit.toMillis(period);  
            // 定时执行  
            new Timer("Lock-Renew-Task").schedule(task, delay, period\_);  
        }  
    }  
   
    static class ScheduleTask extends TimerTask {  
   
        private final RedisDistributionLockPlus redisDistributionLock;  
        private final Map<String, LockContent> lockContentMap;  
   
        public ScheduleTask(RedisDistributionLockPlus redisDistributionLock, Map<String, LockContent> lockContentMap) {  
            this.redisDistributionLock = redisDistributionLock;  
            this.lockContentMap = lockContentMap;  
        }  
   
        @Override  
        public void run() {  
            if (lockContentMap.isEmpty()) {  
                return;  
            }  
            Set<Map.Entry<String, LockContent>> entries = lockContentMap.entrySet();  
            for (Map.Entry<String, LockContent> entry : entries) {  
                String lockKey = entry.getKey();  
                LockContent lockContent = entry.getValue();  
                long expireTime = lockContent.getExpireTime();  
                // 减少线程池中任务数量  
                if ((expireTime - System.currentTimeMillis())/ 1000 < TIME\_SECONDS\_FIVE) {  
                    //线程池异步续约  
                    ThreadPool.submit(() -> {  
                        boolean renew = redisDistributionLock.renew(lockKey, lockContent);  
                        if (renew) {  
                            long expireTimeNew = lockContent.getStartTime() + (expireTime - lockContent.getStartTime()) \* 2 - TIME\_SECONDS\_FIVE \* 1000;  
                            lockContent.setExpireTime(expireTimeNew);  
                        } else {  
                            // 续约失败，说明已经执行完 OR redis 出现问题  
                            lockContentMap.remove(lockKey);  
                        }  
                    });  
                }  
            }  
        }  
    }  
}  



```

通过源码，可以发现：

Redission `加锁`、`解锁`、`续约`都是客户端把一些复杂的业务逻辑，通过封装在`Lua`脚本中发送给`redis`，保证这段复杂业务逻辑执行的`原子性`

有关  **锁过期** 的系统化、体系化的 介绍， 请参见的 专题文章（建议大家看10遍）：

[**史上最全： Redis锁如何续期 ？Redis锁超时，任务没完怎么办？**](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247503521&idx=1&sn=e2f71881eb3abd52d484213089262ebc&scene=21#wechat_redirect)

之四：锁失效 的深坑
----------

redis高可用最常见的方案就是主从复制（master-slave），这种模式也给redis分布式锁挖了一坑。

redis cluster集群环境下，假如现在A客户端想要加锁，它会根据路由规则选择一台master节点写入key mylock，在加锁成功后，master节点会把key异步复制给对应的slave节点。

在Redis主从架构中，写入的，都是 master Redis实例，master 主实例会向 slave 从实例同步key。

一个业务线程 通过向主Redis实例中写入 key-value 来实现加分布式锁，加锁后开始执行业务代码。


一般情况下：如果主master Redis实例挂掉了，会选举出一个从Redis实例成为主的。这是redis 集群的故障转移机制。

但是，如果刚刚加锁的key还没有来得及同步到slave Redis中，新选出的主Redis实例中就没有这个key，这个时候业务线程B就能加锁来获取分布式锁，导致锁失效了。


线程B 加锁成功，也执行业务代码了。

B客户端在新的master节点上加锁成功，而A客户端也以为自己还是成功加了锁的。

此时就会导致同一时间内多个客户端对一个分布式锁完成了加锁，导致各种脏数据的产生。

而这个时候A还没有执行结束，所以就会出现并发安全问题，这就是Redis主从架构下的分布式锁失效问题

本质上，Redis 分布式锁的高可用，有两个层面的解决方案：

Redis 分布式锁的Server端高可用方案, 就是通过配置， 保证Server 尽量可能少的数据丢失。

在redis的配置文件中有两个参数我们可以设置：

```


min-slaves-to-write 1  
min-slaves-max-lag 10  



```

min-slaves-to-write默认情况下是0，min-slaves-max-lag默认情况下是10。

Redis 分布式锁的Server端高可用方案, 就是使用红锁。

在 Java 中，可以使用 Redisson 框架来实现 RedLock。RedissonRedLock 实际上是基于 RedissonMultiLock 实现的，从继承关系可以看出这一点。

Redisson 提供了 RedissonMultiLock 类，它可以同时管理多个锁，并保证操作的原子性。

以下是 Redisson 中 RedLock 的简单使用示例：

```


RedissonClient redisson = // 初始化 Redisson 客户端  
RLock lock1 = redisson.getLock("lock1");  
RLock lock2 = redisson.getLock("lock2");  
RLock lock3 = redisson.getLock("lock3");  
  
RedissonMultiLock multiLock = new RedissonMultiLock(lock1, lock2, lock3);  
try {  
   if (multiLock.tryLock()) {  
       // 成功获取锁，执行业务逻辑  
   } else {  
       // 获取锁失败  
   }  
} finally {  
   multiLock.unlock(); // 释放锁  
}  
  



```

通过以上机制，RedLock 在分布式环境下提供了一种较为可靠的锁方案，能够应对部分节点故障，并保持锁服务的可用性和安全性。

RedLock 具备以下主要特性：

*   互斥性：在任何时间，只有一个客户端可以获得锁，确保了资源的互斥访问。

*   避免死锁：通过为锁设置一个较短的过期时间，即使客户端在获得锁后由于网络故障等原因未能按时释放锁，锁也会因为过期而自动释放，避免了死锁的发生。

*   容错性：即使一部分 Redis 节点宕机，只要大多数节点（即过半数以上的节点）仍在线，RedLock 算法就能继续提供服务，并确保锁的正确性。


有关  **锁失效** 的系统化、体系化的 介绍， 请参见的 专题文章（建议大家看10遍）：

[**史上最全：Redis分布式 锁失效了，怎么办？**](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247503611&idx=1&sn=3cf05137d843f87757f3b7a3c76dbd7b&scene=21#wechat_redirect)

之五：锁分段 的深坑
----------

此外，为了减小锁的粒度，比较常见的做法是将大锁：分段。

java中\`ConcurrentHashMap ，就是将数据分为16段，每一段都有单独的锁，并且处于不同锁段的数据互不干扰，以此来提升锁的性能。

由于秒杀场景的分布式锁，实际上是为了防止超卖， 和库存是强相关的。

所以，可以结合库存，把秒杀的分布式锁 提升性能。

第一步： 把redis 的分段方式进行演进，就是将库存分段分为100段，每一段都有单独的锁，并且处于不同锁段的数据互不干扰，以此来提升锁的性能。。

第二步：使用hash取模法，把用户路由到某一个分段，如果分段里边的库存耗光了，就去访问剩余的总库存。

具体来说，现在的库存中有2000个商品，用户可以秒杀。为了防止出现超卖的情况，通常情况下，可以对库存加锁。如果有1W的用户竞争同一把锁，显然系统吞吐量会非常低。

为了提升系统性能，我们可以将库存分段，比如：分为100段，这样每段就有20个商品可以参与秒杀。

在秒杀的过程中，先把用户id获取hash值，然后除以100取模。

模为1的用户访问第1段库存，模为2的用户访问第2段库存，模为3的用户访问第3段库存，后面以此类推，到最后模为100的用户访问第100段库存。

![](./2025/02/24/redis-锁的5个大坑，如何规避？/5.png)

上面的这个 方案还有一个不足：

*   可能 其他slot槽位 还有 库存， 但是 用户请求 路由到 对应的分片槽位 没有库存， 导致 扣减库存失败


### 解决办法是：在使用hash取模基础上，可以使用动态库存迁移，减少库存消耗不均和无效重试

所以，可以结合库存，把秒杀的分布式锁进行改进。

第一步： 把redis 的分段方式进行演进，额外增加一个总库存分段锁，用于分配存储剩余的总库存。采用多批次少量分配的思路，通过定时任务，从总库存向分段库存中迁移库存。

第二步：使用hash取模法，把用户路由到某一个分段，如果分段里边的库存耗光了，就去访问剩余的总库存。

### 库存动态迁移

为了防止分段多库存耗光，大家都去抢占总库存锁。

采用多批次少量分配的思路，通过定时任务，从总库存向分段库存中迁移库存。

至此， hash取模法的分段锁设计方案，已经完美实现。

并且社群中，已经有小伙伴在生产上完成落地。 以上方案，也是在给他一对一改简历的时候，分享给的。

当然，如果大家简历挖掘不出来亮点，也可以找挖掘， 保证简历金光闪闪、改天换地。

如此一来，在多线程环境中，可以大大的减少锁的冲突。

以前多个线程只能同时竞争1把锁，尤其在秒杀的场景中，竞争太激烈了，简直可以用惨绝人寰来形容，其后果是导致绝大数线程在锁等待。



redis 锁，是一个非常常见的高并发面试题，很多面试官也非常熟悉，上来就让面试者讲讲 redis 锁。

珍藏此文， 帮大家 吊打面试官。

