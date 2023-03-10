---
title: Redis 主从
urlname: bfpwwc
date: '2022-05-07 12:17:50 +0800'
tags:
  - redis
categories:
  - Redis
---

### redis 不能支撑高并发瓶颈在哪里？

> 单机

### redis 要支撑 10 万+ qps，那应该怎么做

> 单机 redis 几乎不太可能 qps 超过 10 万，除非特殊情况，如机器性能特别好，配置特别高
> 单机在几万 qps
> 主从加工>读写分离>支撑 10 万+qps 架构

### 主从架构图解

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1651897982010-6fbf9112-4a3f-4018-bd5c-28d434e8957e.png#averageHue=%23eaeaea&clientId=u9b3c1f56-6522-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=440&id=ue7c876bf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=880&originWidth=999&originalType=binary∶=1&rotation=0&showTitle=false&size=81409&status=done&style=none&taskId=u3da72218-4971-41cf-9a2d-e8572b2a04b&title=&width=499.5)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1651898027917-1c49dbfb-b787-4127-9fff-ec5d2b3168b7.png#averageHue=%23e2e2e2&clientId=u9b3c1f56-6522-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=311&id=u3ad9658a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=621&originWidth=893&originalType=binary∶=1&rotation=0&showTitle=false&size=141468&status=done&style=none&taskId=u41f165fa-739f-411d-930f-c82f9a500bc&title=&width=446.5)

### redis 复制核心机制

1. redis 采用异步方式复制数据到 slave 节点上，从 2.8 后 slave node 会周期性确认自己复制的数据量。
2. 一个 master node 可以配置多个 slave node。
3. slave node 可以连接其他 slave node。
4. slave node 做复制时是不会影响 master node 正常工作的。
5. slave node 做复制时不会影响自己的查询操作，他会用旧的数据集来提供服务；复制完成后，需要删除旧数据，加载新数据集，这时候就会暂停对外服务了。
6. slave node 主要进行横向扩容，做读写分离，扩容的 slave node 会提高吞吐量。

### master 持久化对主从架构的意义

1. 如果采用主从架构，必须开启 master node 持久化
2. 不建议用 slave node 作为 master node 的热备，如果你关掉 master node 持久化，master node 宕机重启后数据是空的，同步到 slave node 数据也丢了

### 主从复制原理

#### 主从配置

reids 支持主从复制功能，可以通过 slaveof（redis 5 后改为 replicaof）或者在配置文件中设置 slaveof 来开启复制。（主对外从对内，主可写，从不可写，主宕机，从不可为主）
配置：

```bash
slaveof <masterip> <masterport># redis5 之前
replicaof <masterip> <masterport># redis5 之后
```

###### 作用

1. 读写分离
   一主多从，主从同步。主负责写，从负责读。提升 redis 性能和吞吐量，主从一致性问题
2. 数据容灾
   从机是主机的备份，主机宕机，从机可读不可写。默认情况下主机宕机，从机不可为主，利用哨兵可以实现主从切换，做到高可用。

#### 原理和实现

1. 保存节点信息
2. 建立 socket 连接
3. 发送 ping 命令
4. 权限认证
5. 同步数据
6. 命令传播

#### 心跳检测

在命令传播阶段，从服务器默认每秒向主服务器发送命令：

```bash
replconf ack <replication_offset>
#ack :应答
#replication_offset：从服务器当前的复制偏移量
```

##### 作用

1. 检测主从连接状态
2. 辅助实现 min-slave
3. 检测命令丢失

####

#### 主从架构核心原理

> 当启动一个 slave node 时，他会发送一个 PSYNC 命令给 master node
> 如果 slave node 是第一次连接 master node 则会触发一次 full resynchronization （全量备份）
> 如果非第一次连接 master node 则会复制给 slave node 缺少的数据

开始 full resynchornization 时，master 会启动一个后台线程，开始生成一份 RDB 快照文件，同时还会将从客户端接受到的所有写命令缓存到内存中。RDB 快照生成后，master 会将 RDB 发送给 slave node，slave 会先写入本地磁盘，然后从本地磁盘加载到内存中，然后 master 会将内存中的写命令发送给 slave，slave 同步写数据

#### 主从复制断点续传

> 从 2.8 后，支持主从复制断点续传，如果主从复制过程中，网络连接突然断掉，那么可以接着上次复制的地方继续复制而不用从头开始复制。
> 原理：master node 中有一个 backlog，该文件保存了 master 和 slave replaca offset 和 master id，如果 master 和 slave 网络突然断掉，slave 会从上次 标记的 offset 开始继续复制。
> 如果没有找到 offset 则会触发 full resynchornizaton

#### 复制完整流程

1. slave node 启动，仅仅保存了 master node 信息，包括 master 的 host 和 ip，但是复制还没开始 master host 和 ip 从哪里获取的？ 答案：从 redis.conf 里面的 slaveof 配置的
2. slave node 内部有个定时任务，每秒检查是否有新的 master node 要连接和复制，如果发现就跟 master node 建立 socket 网络连接
3. slave node 发送 ping 命令给 master node
4. 口令认证：如果 master node 设置了 requirepass ，那么 slave node 必须发送 masterauth 口令过去认证
5. master node 第一次进行全量复制，会将所有数据发送给 slave node
6. master node 后续持续将写命令，异步复制给 slave node

#### 数据同步核心机制

1. master 和 slave 都会各自维护一个 offset
   master 会在自身不断累积 offset，slave 也会在自身不断累加 offset，slave node 每秒都会上报自己的 offset 给 master ，同时 master 会保存每个 slave node 的 offset。
2. backlog
   master node 有一个 blacklog ，默认大小 1M，master node 给 slave node 复制数据时，也会将数据在 backlog 中写一份，backlog 主要用在做全量备份中断后的增量复制的。
3. master run id
   master run id 如果根据 host+ip 定位，是不靠谱的；如果 master node 重启或者数据出现了变化，
   slave 应该根据不同的 run id 进行区分，run id 不同就做全量备份
   如果需要不更改 run id 重启 redis，可以使用 redis-cli debug reload 命令
4. psync
   slave node 使用 psync 从 master node 进行复制，psync runid offset master node 会根据自身情况返回响应信息，可能是触发全量复制，也可能触发增量复制。

#### 全量复制

1. master 执行 bgsave ，在本地生成一份 RDB 快照文件
2. master node 会将 RDB 快照文件发送给 slave node，如果 rdb 复制文件超过 60 s（repl-timeout），那么 slave node 则会认为复制失败，可以适当调节该参数
3. 对于千兆网卡机器，一般每秒传输 100MB，6G 文件可能超过 60s。
4. master node 生成 RDB 快照时，会将所有的写操作缓存到内存中，在 slave node 保存 RDB 快照后再将写命令复制给 slave node。
5. client-output-buffer-limit slave 256MB 64MB 60 如果在复制期间，内存缓冲区持续消耗超过 64MB，或者一次性超过 256MB，那么停止复制，复制失败
6. slave node 接收到 rdb 之后，清空自己的旧数据，然后重新加载 rdb 到自己的内存中，同时基于旧的数据版本对外提供服务
7. 如果 slave node 开启了 AOF，那么会立即执行 BGREWRITEAOF，重写 AOF

#### heartbeat

主从节点都会互相发送 heartbeat 信息，master node 每隔 10s 发送一次 heartbeat，slave node 每隔 1s 发送一次。

### slave 如何处理过期 key

> slave 不会过期 key ，如果 master 过期了一个 key 那么会模拟一条 del 命令发送给 slave

### reids 不可用是什么？单例不可用？主从架构不可用？不可用后果是什么？

#### redis 不可用是什么？

1. redis 进程死了。
2. redis 所在机器死了。

#### 单例不可用？主从架构不可用？

1. 一个 slave 挂掉，是不会影响可用性的，还有其他的 slave 提供相同数据下的对外服务。
2. 如果是 master node 挂掉，就没办法写数据了，slave node 就没有用了，因为没有 master node 给他们写数据了，这是系统相当于就是不可用了。

#### 不可用后果是什么？

1. 高并发高性能的缓存不能用了，超过 mysql 最大承载能力大并发的大流量会涌入 mysql ，进而导致 mysql 宕机。

### Redis 高可用架构,主备切换

> redis 高可用架构又叫故障转移 failover,也叫主备切换
> 在 master node 故障时自动检测,并且将某个 slave node 切换为 master node
> 依赖于 sentinal node 哨兵

#### 哨兵

##### 哨兵介绍

sentinel 中文名哨兵
主要功能:

1. 集群监控,负责监控 master 和 slave
2. 消息通知,如果某个 redis 实例有故障,那么哨兵负责发送消息作为报警给管理员
3. 故障转移,如果 master node 挂掉会自动转移到 slave node 上
4. 配置中心,如果故障转移发生了,通知 clinet 客户端新的 master 地址

涉及到的问题:

1. 分布式选举问题,如果故障转移时判断一个 master node 故障时,需要大部分哨兵同意才行
2. 即使部分哨兵节点挂掉了,哨兵集群还是能正常工作的

##### 哨兵核心知识

1. 哨兵至少需要三个实例,保证自己的健壮性
2. 哨兵+redis 主从架构,不会保证数据零丢失,只能保证 redis 集群高可用性

##### 为什么 redis 哨兵集群只有两个节点无法工作？

哨兵集群必须部署两个节点以上
如果哨兵集群仅部署了 2 个哨兵实例，quorum = 1
master 宕机，s1 和 s2 中只要有 1 个哨兵认为 master 宕机就可以还行切换，同时 s1 和 s2 中会选举出一个哨兵来执行故障转移
同时这个时候，需要 majority，也就是大多数哨兵都是运行的，2 个哨兵的 majority 就是 2（2 的 majority=2，3 的 majority=2，5 的 majority=3，4 的 majority=2），2 个哨兵都运行着，就可以允许执行故障转移
但是如果整个 M1 和 S1 运行的机器宕机了，那么哨兵只有 1 个了，此时就没有 majority 来允许执行故障转移，虽然另外一台机器还有一个 R1，但是故障转移不会执行

##### redis 如何解决哨兵主备切换数据丢失问题?

###### 丢失的场景

1. 异步复制导致的丢失; 因为 master->slave 复制是异步的,部分数据没有复制到 slave 上 master 就宕机了,此时这些部分数据就丢失
2. 脑裂导致的丢失；master 所在的网络突然脱离了正常网络，跟其他 slave 机器不能连接，但 master 还在运行，但其他的 slave 认为 master 宕机了，并开启选举，将其他 slave 换成了 master，这时集群中就有了两个 master 也就是脑裂，此时某个 slave 被切换成 master，但可能 client 还没有切换到 master ，还继续写向 master 的数据可能也丢失了，此时旧的 master 在回复过来时，会被作为新的 slave 挂到新 master 上去，自己的数据会清空，重新从新的 master 复制数据

###### 解决方法

修改配置：
至少要求一个 slave ：min-slaves-to-write 1
数据同步和延迟不超过 10 s ：min-slaves-max-lag 10

1. min-slaves-max-lag 可以减少异步复制导致的数据丢失，
   一旦 slave 复制数据和 ack 延时太长，就认为可能 master 宕机后损失的数据太多了，那么就拒绝写请求，这样可以把 master 宕机时由于部分数据未同步到 slave 导致的数据丢失降低到可控范围
2. 减少脑裂丢失数据，上面两个配置说，如果不能给指定数量的 slave 发送数据，而且 slave 超过 10 s 没有给自己 ack 消息，那么就直接拒绝客户端写请求

### redis 多个底层原理深入解析

#### sdown 和 odown 转换机制

sdown 和 odown 两种失败状态

1. sdown 是主观宕机，就一个哨兵如果自己觉得一个 master 宕机了，那么就是主观宕机。
   达成条件：如果一个哨兵 ping 一个 master ，超过 is-master-down-after-milliseconds 指定的毫秒数之后，就主观认为 master 宕机。
2. odown 是客观宕机，如果 quorum 数量的哨兵都觉得一个 master 宕机了，那么就是客观宕机。
   如果一个哨兵收到 quorum 数量的其他哨兵也认为那个 master 是 sdown 了，那么认为 odown，客观认为 master 宕机。

#### 哨兵集群自动发现机制

1. 哨兵互相之间的发现，是通过 redis pub/sub（消息订阅）系统实现的，每个哨兵都会往 **sentinel**:hello 这个 channel 里发送一个消息，这时候其他哨兵都可以消费到这个消息，并感知到其他哨兵的存在。
2. 每隔 2 s ，每个哨兵都会往自己监控的某个 master+slaves 对应的 **sentinel**:hello channel 发送一个消息，内容是自己的 host、ip、runid 和对这个 master 的配置。
3. 每个哨兵也会监听自己监控每个 master+slave 对应的 **sentinel**:hello channel,然后去感知同伴监听这个 master+slaves 的其他哨兵的存在
4. 每个哨兵还会跟其他哨兵交换对 master 的监控配置，互相进行监控配置同步。

#### slave 配置自动纠正

哨兵会负责自动纠正 slave 的一些配置，比如

1. slave 如果要成为潜在的 master 候选人，哨兵会确保 slave 在复制现有 master 的数据。
2. 如果 slave 连接到一个错误的 master 上，比如故障转移之后，那么哨兵会确保他们连接到正确的 master 上。
