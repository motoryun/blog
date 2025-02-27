---
title: 最新Seata集成了RocketMQ事务消息yyds
date: 2025-02-27 11:03:11
tags: Seata集成RocketMQ
categories: 架构
---


****\-** 一：回顾Seat AT 模式** 

 - AT模式角色如下

- 分支事务 处理逻辑如下

- AT模式第一阶段  Prepare 阶段

#####     -这也是Seata和XA事务的不同之处：

- AT模式第二阶段

#####     -场景一：提交，全局提交

#####     -场景二：回滚，全局回滚

- AT模式相对于XA模式的优势

- 秒杀实操的AT分布式事务架构

- Seat AT 模式的不足

****\-**** **二：回顾Seat TCC 模式** 

 - Seata TCC基本原理

- Seata TCC模式的流程图

- Seata TCC 事务的3个操作

- TCC 模式第一阶段  try 阶段

- TCC 模式第二阶段  

 - TCC版本的秒杀的分布式事务架构

- Seata TCC 事务的弱点

****\-**** **三：新功能  Seata + Rocketmq 事务消息实现 强弱结合型事务**

- Seata  + Rocketmq 事务消息 结合

- Seata  + Rocketmq 事务消息 结合的使用场景

****\-**** **说在最后：有问题找老架构取经**

一：回顾Seat AT 模式
--------------

Seata AT模式是基于XA事务演进而来的一个分布式事务中间件，

XA是一个基于数据库实现的分布式事务协议，本质上和两阶段提交一样，需要数据库支持，Mysql5.6以上版本支持XA协议，其他数据库如Oracle，DB2也实现了XA接口

#### AT模式角色如下

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img.png)

1、Transaction Coordinator (TC)：事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚

2、Transaction Manager ™：

控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议

3、Resource Manager (RM)：

控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚

来自官方的图：

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_1.png)

#### 分支事务 处理逻辑如下

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_2.png)

Branch就是指的分布式事务中每个独立的本地局部事务

#### AT模式第一阶段 Prepare 阶段

看看Prepare 阶段 简单的图

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_3.png)

Seata 的 JDBC 数据源代理通过对业务 SQL 的解析，把业务数据在更新前后的数据镜像组织成回滚日志，利用 本地事务 的 ACID 特性，将业务数据的更新和回滚日志的写入在同一个 本地事务 中提交。

这样，可以保证：**任何提交的业务数据的更新一定有相应的回滚日志存在**

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_4.png)

基于这样的机制，分支的本地事务便可以在全局事务的第一阶段提交，并马上释放本地事务锁定的资源

###### 这也是Seata和XA事务的不同之处：

经典的2PC两阶段提交（XA）往往对**资源的锁定需要持续到第二阶段实际的提交或者回滚操作**，

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_5.png)

AT模式，可以在第一阶段释放对资源的锁定，降低了锁范围

> 谁的功劳：回滚日志

#### AT模式第二阶段

看看 commit/rollback 阶段 简单的图

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_6.png)

**两种情况**

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_7.png)

##### 场景一：提交，全局提交

如果决议是**全局提交**，此时分支事务此时已经完成提交，不需要同步协调处理（只需要异步清理回滚日志），Phase2 可以非常快速地完成
![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_8.png)

##### 场景二：回滚，全局回滚

如果决议是全局回滚，RM 收到协调器发来的回滚请求，通过 XID 和 Branch ID 找到相应的回滚日志记录，**通过回滚记录生成反向的更新 SQL 并执行**，以完成分支的回滚

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_9.png)

#### AT模式相对于XA模式的优势

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_10.png)

提高效率，即使第二阶段发生异常需要回滚，只需找对undolog中对应数据并反解析成sql来达到回滚目的

同时Seata无入侵，通过代理数据源将业务sql的执行解析成undolog来与业务数据的更新同时入库，达到了对业务无侵入的效果

#### 秒杀实操的AT分布式事务架构

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_11.png)

Seat AT 模式的不足
-------------

AT模式的依赖的还是依赖本地数据源的事务控制能力，完成分支事务的管理。

AT模式 采用的是wal的思想，提交事务的时候同时记录undolog，如果全局事务成功，则删除undolog，如果失败，则使用undolog的数据回滚分支事务，最后删除undolog。

Seat AT 模式的性能低，而且非常低。

二：回顾Seat TCC 模式
===============

Seat TCC 模式 的性能会高些， 脱离了本地数据库事务， 需要业务控制 两个阶段操作的 原子性。

### Seata TCC基本原理

TCC模式的特点是不再依赖于undolog，

#### Seata TCC模式的流程图

TCC模式还是采用2阶段提交的方式：

*   第一阶段使用prepare尝试事务提交，

*   第二阶段使用commit或者rollback让事务提交或者回滚。


引用网上一张TCC原理的参考图片
![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_12.png)

来自官方的图：

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_13.png)

#### Seata TCC 事务的3个操作

TCC 将事务提交分为 Try - Confirm - Cancel 3个操作。

其和两阶段提交有点类似，Try为第一阶段，Confirm - Cancel为第二阶段，是一种应用层面侵入业务的两阶段提交。

| **操作方法** | **含义** |
| --- | --- |
|Try| 预留业务资源/数据效验|
| Confirm| 确认执行业务操作，实际提交数据，不做任何业务检查，try成功，confirm必定成功，需保证幂等|
| Cancel| 取消执行业务操作，实际回滚数据，需保证幂等|

其核心在于将业务分为两个操作步骤完成。不依赖 RM 对分布式事务的支持，而是通过对业务逻辑的分解来实现分布式事务。

下面还以银行转账例子来说明

假设用户user表中有两个字段：可用余额(available\_money)、冻结余额(frozen\_money)

*   A扣钱对应服务A(ServiceA)

*   B加钱对应服务B(ServiceB)

*   转账订单服务(OrderService)

*   业务转账方法服务(BusinessService)


ServiceA，ServiceB，OrderService都需分别实现try()，confirm()，cancle()方法，方法对应业务逻辑如下

|    | **ServiceA** | **ServiceB** | **OrderService** |
| --- | --- | --- | --- |
| try()| 校验余额(并发控制) 冻结余额+1000 余额-1000| 冻结余额+1000| 创建转账订单，状态待转账|
| confirm()| 冻结余额-1000| 余额+1000 冻结余额-1000| 状态变为转账成功|
| cancle()| 冻结余额-1000 余额+1000| 冻结余额-1000| 状态变为转账失败|

其中业务调用方BusinessService中就需要调用

*   ServiceA.try()

*   ServiceB.try()

*   OrderService.try()


1、当所有try()方法均执行成功时，对全局事物进行提交，即由事物管理器调用每个微服务的confirm()方法 2、 当任意一个方法try()失败(预留资源不足，抑或网络异常，代码异常等任何异常)，由事物管理器调用每个微服务的cancle()方法对全局事务进行回滚

#### TCC 模式第一阶段 try 阶段

看看尼恩给大家画的 try 阶段 简单的图

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_14.png)

try 阶段 没有本地事务了。需要业务维护 本地操作的 原子性，一致性。

#### TCC 模式第二阶段 

看看尼恩给大家画的第二阶段 简单的图

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_15.png)

TCC 模式 第二 阶段 也没有本地的undo 日志支持。需要业务维护 完成 本地 的提交操作， 或者 回滚操作。

而 AT模式的话， 本地 的提交操作， 或者 回滚操作 是 proxy 代理结合 undo 日志 完成的， 是业务无感知的， 无入侵的。

### TCC版本的秒杀的分布式事务架构

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_16.png)

### Seata TCC 事务的弱点

Seata TCC 事务脱离了对数据库的强依赖， 很明确来说，性能会高些。

但是 Seata TCC 事务的弱点不少，大致如下：

*   需要自定义 提交操作 + 回滚操作， 存在业务入侵。

*   需要处理 悬挂 问题

*   需要处理 空回滚问题


三：新功能 Seata + Rocketmq 事务消息实现 强弱结合型事务
-------------------------------------

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_17.png)

首先看看 RocketMQ 的事务消息， 能够保证本地操作 + 消息发送的原子性。

具体来说， 主要是保证了本地方法执行和消息发送在一个分布式事务中，要不全部成功，要不全部失败。

见下图：

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_18.png)

RocketMQ 通过发送 half 消息来实现，下面详细说明一下：

1.  消息发送方 向 Broker 发送一条 half 消息；

2.  half 消息发送成功后，消息发送方 执行本地事务；

3.  如果 消息发送方 执行本地事务成功，则向 Broker 发送 commit 请求，否则发送 rollback 请求；

4.  如果 Broker 收到的是 rollback 请求，则删除保存的 half 消息；

5.  如果 Broker 收到的是 commit 请求，则把 half 消息投递到 真实 队列， 等待消费服务来拉取，然后删除保存的 half 消息；

6.  如果 Broker 没有收到 rollback/commit 请求，则会发送请求到 Producer 查询本地事务状态，然后根据 Producer 返回的本地状态做 commit/rollback 相关处理。


Seata + Rocketmq 事务消息 结合
------------------------

Seata + Rocketmq 事务消息 结合的目标：

*   一 是保证 分布式事务 + 消息 发送的原子性。

*   二 是 再通过mq的重试机制，去保证订阅者的最终一致性，


![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_19.png)

Seata 的改进主要在 prepare 阶段。

Seata 提供了一个 SeataMQProducer 类，把 RocketMQ 中 TransactionListener 的方法加入到全局事务。

见下面代码：

```


SeataMQProducer(final String namespace, final String producerGroup, RPCHook rpcHook) {  
 super(namespace, producerGroup, rpcHook);  
 this.transactionListener = new TransactionListener() {  
  @Override  
  public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {  
   return LocalTransactionState.UNKNOW;  
  }  
  
  @Override  
  public LocalTransactionState checkLocalTransaction(MessageExt msg) {  
   String xid = msg.getProperty(PROPERTY\_SEATA\_XID);  
   if (StringUtils.isBlank(xid)) {  
    LOGGER.error("msg has no xid, msgTransactionId: {}, msg will be rollback", msg.getTransactionId());  
    return LocalTransactionState.ROLLBACK\_MESSAGE;  
   }  
   GlobalStatus globalStatus = DefaultResourceManager.get().getGlobalStatus(SeataMQProducerFactory.ROCKET\_BRANCH\_TYPE, xid);  
   if (COMMIT\_STATUSES.contains(globalStatus)) {  
    return LocalTransactionState.COMMIT\_MESSAGE;  
   } else if (ROLLBACK\_STATUSES.contains(globalStatus) || GlobalStatus.isOnePhaseTimeout(globalStatus)) {  
    return LocalTransactionState.ROLLBACK\_MESSAGE;  
   } else if (GlobalStatus.Finished.equals(globalStatus)) {  
    LOGGER.error("global transaction finished, msg will be rollback, xid: {}", xid);  
    return LocalTransactionState.ROLLBACK\_MESSAGE;  
   }  
   return LocalTransactionState.UNKNOW;  
  }  
 };  



```

在 prepare 阶段，SeataMQProducer 向 RocketMQ Broker 发送 half 消息，执行本地事务，如果执行成功，则强行把 LocalTransactionState 改回 UNKNOW，等待 TC 发送指令，决定是 commit 或 rollback。

如果执行失败，返回 ROLLBACK\_MESSAGE，TC 下发指令，回滚全局事务。

Seata + Rocketmq 事务消息 结合的使用场景
-----------------------------

如果 MQ Broker 没有收到 commit/rollback 消息，则会回查 Producer 本地事务状态，也就是上面代码中的 checkLocalTransaction。

checkLocalTransaction 检查 全局事务状态，使用 XID 去查询 全局事务，去决定 half 消息 是 抛弃还是 投递 。

集成 RocketMQ 之后，Seata 的分布式事务调用流程， 下面以 订单服务、库存服务两个服务为例：

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_20.png)

Apache Seata 引入 RocketMQ 后，支持的分布式事务场景更加丰富，使得 Seata 可以用于 强一致性 + 弱一致性 结合的场景。

![](./2025/02/27/最新Seata集成了RocketMQ事务消息yyds/img_21.png)
