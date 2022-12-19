---
title: Redis 缓存淘汰策略
urlname: un6niq
date: '2022-05-07 12:17:47 +0800'
tags:
  - redis
categories:
  - Redis
---

### redis 有哪些缓存淘汰策略?

> redis 数据过期策略采用定期删除+惰性删除

1. volatile-LRU least recently used 优先删除最近最少使用,只针对设置了 expire 过期时间的 key 生效
2. allkeys-LRU 所有 key 通用,优先删除最近最少使用的 key
3. volatile-LFU 优先删除最不常用的 key
4. random 随机删除一部分 key
5. ttl 优先删除剩余时间最短的 key

### 缓存过期

#### maxmemory

maxmemory：默认设置为 0 不限制。
问题：超过物理内存后性能急剧下降，甚至崩溃，硬盘与内存交换（swap）虚拟内存，频繁 IO 性能急剧下降。当趋近 maxmemory 时，通过缓存淘汰策略从内存中删除对象。
设置方式：
在 redis.conf 中

```bash
maxmemory 1024mb

#获取配置命令
config get maxmemory
```

#### expire

使用 expire 设置一个键的存活时间（ttl：time to live），过了这段时间，该键会自动被删除

```bash
expire key 2 #2s 后失效
ttl key #查看失效状态，负数为永久失效
```

### 淘汰策略

#### 定时删除

创建定时器，设置键过期时间同时，立即对键执行删除操作，消耗 cpu ，不推荐

#### 惰性删除

读取的时候，检查是否失效，失效就删除，调用 expireIfNeeded

#### 主动删除

在 reids.conf 中配置主动删除策略，默认是 no-enviction（不删除）。

```bash
maxmemory-policy 淘汰策略
```

##### LRU

LRU（least recently used）最近最少使用，算法根据数据的历史访问记录来进行淘汰数据，其核心思想是‘如果数据最近被访问过，那么将来被访问的几率也更高’

```markdown
### 实现策略

1. 新数据插入到链表头部；
2. 每当缓存命中时（即缓存数据被访问），则将数据移动到链表头部；
3. 当链表满的时候，将链表尾部数据丢弃。
4. 在 java 中可以使用 LinkHashMap（哈希链表）去实现 LRU。
   在服务器配置中保存了 lru 计数器 server.lrulock，会定时（redis 定时程序 serverCorn()）更新，server.lrulock 的值是根据 server.unixtime 计算出来的。另外，从 struct redisObject 中可以发现，每一个 redis 对象都会设置相应的 lru。可以想象的是，每一次访问数据的时候，会更新 redisObject.lru。
   LRU 数据淘汰机制是这样的：在数据集中随机挑选几个键值对，取出其中 lru 最大的键值对淘汰。不可能遍历 key 用当前时间-最近访问 越大 说明 访问间隔时间越长
```

1. volatile-lru： 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰。
2. allkeys-lru：从数据集中挑选最近最少使用数据淘汰。

##### RANDOM

1. volatile-random：从设置过期时间的数据集中随机选择数据淘汰
2. allkeys-random：从数据集中随机选取数据淘汰

#### TTL（time to live）

volatile-ttl：从设置过期时间的数据集中挑选将要过期的数据淘汰
TTL 淘汰机制：从过期时间的表中随机挑选几个键值对，取出其中 ttl 最小的键值对淘汰。

#### noenviction

禁止驱逐数据，不删除 默认
