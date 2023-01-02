---
title: Redis 分区
urlname: rcfekk
date: '2022-05-07 12:25:46 +0800'
tags:
  - redis
categories:
  - Redis
---

## 1 分区的意义与方式

分区是将数据分布在多个 Redis 实例（Redis 主机）上，以至于每个实例只包含一部分数据。
**1.1 意义**

- 性能的提升

单机 Redis 的网络 I/O 能力和计算资源是有限的，将请求分散到多台机器，充分利用多台机器的计算能力。可网络带宽，有助于提高 Redis 总体的服务能力。

- 存储能力的横向扩展

即使 Redis 的服务能力能够满足应用需求，但是随着存储数据的增加，单台机器受限于机器本身的存储容量，将数据分散到多台机器上存储使得 Redis 服务可以横向扩展。
**1.2 方式**

- 范围分区

好处：实现简单，方便迁移和扩展 缺陷：热点数据分布不均，性能损失。非数字型 key，比如 uuid 无法使用(可采用雪花算法替代)。分布式环境主键雪花算法是数字能排序

- hash 分区

利用简单的 hash 算法即可：Redis 实例=hash(key)%N。key:要进行分区的键，比如 user_id，N:Redis 实例个数(Redis 主机)
好处：支持任何类型的 key，热点分布较均匀，性能较好 缺陷：迁移复杂，需要重新计算，扩展较差（利用一致性 hash 环）

## 2 client 端分区

对于一个给定的 key，客户端直接选择正确的节点来进行读写。许多 Redis 客户端都实现了客户端分区(JedisPool)，也可以自行编程实现。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669880694-741ba555-b8cb-4e30-a901-d61c12528733.png#averageHue=%23f7f4ed&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uce07fe18&margin=%5Bobject%20Object%5D&name=image.png&originHeight=547&originWidth=580&originalType=url∶=1&rotation=0&showTitle=false&size=156039&status=done&style=none&taskId=u3340474f-2d10-4ac5-b0cb-98c50286f92&title=)
**客户端选择算法：**

- **hash**

普通 hash，hash(key)%N，hash:可以采用 hash 算法，比如 CRC32、CRC16 等，N:是 Redis 主机个数
user_id : u001 hash(u001) : 1844213068 Redis 实例=1844213068%3 余数为 2，所以选择 Redis3。
普通 Hash 的优势，实现简单，热点数据分布均匀。
普通 Hash 的缺陷，节点数固定，扩展的话需要重新计算查询时必须用分片的 key 来查，一旦 key 改变，数据就查不出了，所以要使用不易改变的 key 进行分片

- **一致 hash 算法**

普通 hash 是对主机数量取模，而一致性 hash 是对 2^32（4 294 967 296）取模。我们把 2^32 想象成一个圆，就像钟表一样，钟表的圆可以理解成由 60 个点组成的圆，而此处我们把这个圆想象成由 2^32 个点组成的圆。
优点：添加或移除节点时，数据只需要做部分的迁移，而其他的数据保持不变。添加效果是一样的。
问题：hash 环偏移，在介绍一致性哈希的概念时，我们理想化的将 3 台服务器均匀的映射到了 hash 环上。也就是说数据的范围是 2^32/N。但实际情况往往不是这样的。有可能某个服务器的数据会很多，某个服务器的数据会很少，造成服务器性能不平均。这种现象称为 hash 环偏移。可以通过"虚拟节点"是"实际节点"（实际的物理服务器）在 hash 环上的复制品,一个实际节点可以对应多个虚拟节点。
缺点：复杂度高客户端需要自己处理数据路由、高可用、故障转移等问题使用分区，数据的处理会变得复杂，不得不对付多个 redis 数据库和 AOF 文件，不得在多个实例和主机之间持久化你的数据。不易扩展，一旦节点的增或者删操作，都会导致 key 无法在 redis 中命中，必须重新根据节点计算，并手动迁移全部或部分数据。
**3 proxy 端分区**
在客户端和服务器端引入一个代理或代理集群，客户端将命令发送到代理上，由代理根据算法，将命令路由到相应的服务器上。常见的代理有 Codis（豌豆荚）和 TwemProxy（Twitter）。
**部署架构：**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669880568-c94f044c-faac-4571-b512-c4449d3b8a32.png#averageHue=%23f8f8f8&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u66db5a3a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=399&originWidth=760&originalType=url∶=1&rotation=0&showTitle=false&size=97338&status=done&style=none&taskId=u5c891d49-0674-413e-96fb-4cef469e402&title=)
**组件组成：**
**Codis Server**：基于 redis-3.2.8 分支开发。增加了额外的数据结构，以支持 slot 有关的操作以及数据迁移指令。
**Codis Proxy**：客户端连接的 Redis 代理服务, 实现了 Redis 协议。 除部分命令不支持以外(不支持的命令列表)，表现的和原生的 Redis 没有区别（就像 Twemproxy）。对于同一个业务集群而言，可以同时部署多个 codis-proxy 实例；不同 codis-proxy 之间由 codis-dashboard 保证状态同步。
**Codis Dashboard**：集群管理工具，支持 codis-proxy、codis-server 的添加、删除，以及据迁移等操作。在集群状态发生改变时，codis-dashboard 维护集群下所有 codis-proxy 的状态的一致性。对于同一个业务集群而言，同一个时刻 codis-dashboard 只能有 0 个或者 1 个；所有对集群的修改都必须通过 codis-dashboard 完成。
**Codis Admin**：集群管理的命令行工具。可用于控制 codis-proxy、codis-dashboard 状态以及访问外部存储。
**Codis FE**：集群管理界面。多个集群实例共享可以共享同一个前端展示页面；通过配置文件管理后端 codis-dashboard 列表，配置文件可自动更新。
**Storage**：为集群状态提供外部存储。提供 Namespace 概念，不同集群的会按照不同 product name 进行组织；目前仅提供了 Zookeeper、Etcd、Fs 三种实现，但是提供了抽象的 interface 可自行扩展。
**分区原理：**
Codis 将所有的 key 默认划分为 1024 个槽位(slot)，它首先对客户端传过来的 key 进行 crc32 运算计算哈希值，再将 hash 后的整数值对 1024 这个整数进行取模得到一个余数，这个余数就是对应 key 的槽位。Codis 的槽位和分组的映射关系就保存在 codis proxy 当中。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669880594-66bda482-d24f-4047-b420-64fb59faac4f.png#averageHue=%23f1f1f1&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uc3779fa2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=342&originWidth=763&originalType=url∶=1&rotation=0&showTitle=false&size=74381&status=done&style=none&taskId=uf73fac4c-cda2-4347-9851-d18a9d535f5&title=)
实例之间槽位同步：
codis proxy 存在单点问题，需要做集群。不同 proxy 之间需要同步映射关系，在 Codis 中使用的是 Zookeeper（Etcd）来保存映射关系
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669880628-7cbfc913-00b5-4bec-879d-8615a6a967d9.png#averageHue=%23fbfaf8&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u90580fb6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=361&originWidth=780&originalType=url∶=1&rotation=0&showTitle=false&size=107854&status=done&style=none&taskId=u404ae460-63dd-4b45-a8cb-68d1a3a2762&title=)
Codis 将槽位关系存储在 zk 中，并且提供了一个 Dashboard 可以用来观察和修改槽位关系，当槽位关系变化时，Codis Proxy 会监听到变化并重新同步槽位关系，从而实现多个 Codis Proxy 之间共享相同的槽位关系配置。
**优点：**
对客户端透明,与 codis 交互方式和 redis 本身交互一样，支持在线数据迁移,迁移过程对客户端透明有简单的管理和监控界面，支持高可用,无论是 redis 数据存储还是代理节点，自动进行数据的均衡分配。最大支持 1024 个 redis 实例,存储容量海量高性能
**缺点：**
采用自有的 redis 分支,不能与原版的 redis 保持同步，如果 codis 的 proxy 只有一个的情况下, redis 的性能会下降 20%左右。某些命令不支持

## 4 Redis cluster 分区

Redis3.0 之后，Redis 官方提供了完整的集群解决方案。方案采用去中心化的方式，包括：sharding（分区）、replication（复制）、failover（故障转移）。称为 RedisCluster。Redis5.0 前采用 redis-trib 进行集群的创建和管理，需要 ruby 支持。Redis5.0 可以直接使用 Redis-cli 进行集群的创建和管理

### **4.1 部署架构**

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669880752-0e310ca6-ce37-4b13-b0c9-31271231f777.png#averageHue=%23f7f3f1&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf64b028c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=466&originWidth=755&originalType=url∶=1&rotation=0&showTitle=false&size=198832&status=done&style=none&taskId=ubc054809-65a2-42a8-b9e9-6324eef1b72&title=)

### **4.1.1 去中心化**

RedisCluster 由多个 Redis 节点组构成，是一个 P2P 无中心节点的集群架构，依靠 Gossip 协议传播的集群。

### **4.1.2 Gossip 协议**

Gossip 协议是一个通信协议，一种传播消息的方式。起源于：病毒传播。Gossip 协议基本思想就是：一个节点周期性(每秒)随机选择一些节点，并把信息传递给这些节点。这些收到信息的节点接下来会做同样的事情，即把这些信息传递给其他一些随机选择的节点。信息会周期性的传递给 N 个目标节点。这个 N 被称为**fanout**（扇出）gossip 协议包含多种消息，包括 meet、ping、pong、fail、publish 等等。通过 gossip 协议，cluster 可以提供集群间状态同步更新、选举自助 failover 等重要的集群功能
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669881997-d349f3f4-b261-4714-b814-e5a05479a54c.png#averageHue=%23f1f1f1&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u34b2c726&margin=%5Bobject%20Object%5D&name=image.png&originHeight=270&originWidth=764&originalType=url∶=1&rotation=0&showTitle=false&size=177372&status=done&style=none&taskId=u4c2d0913-f093-48d6-97e6-64516c163d9&title=)

### **4.1.3 slot**

redis-cluster 把所有的物理节点映射到[0-16383]个**slot**上,基本上采用平均分配和连续分配的方式。比如上图中有 5 个主节点，这样在 RedisCluster 创建时，slot 槽可按下表分配：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669882212-a016dcc1-0740-41fb-ad29-432b5804ac38.png#averageHue=%23f9f9f8&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub6f5f5a0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=243&originWidth=772&originalType=url∶=1&rotation=0&showTitle=false&size=78562&status=done&style=none&taskId=ubfab1ccd-654e-47c4-929c-e76f076dbf8&title=)
cluster 负责维护节点和 slot 槽的对应关系 value------>slot-------->节点。当需要在 Redis 集群中放置一个 key-value 时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点。比如：set name zhaoyun；hash("name")采用 crc16 算法，得到值：1324203551%16384=15903，根据上表 15903 在 13088-16383 之间，所以 name 被存储在 Redis5 节点。slot 槽必须在节点上连续分配，如果出现不连续的情况，则 RedisCluster 不能工作，详见容错。

### **4.1.4 RedisCluster 的优势**

- 高性能

Redis Cluster 的性能与单节点部署是同级别的。多主节点、负载均衡、读写分离

- 高可用

Redis Cluster 支持标准的 主从复制配置来保障高可用和高可靠。failover，Redis Cluster 也实现了一个类似 Raft 的共识方式，来保障整个集群的可用性。

- 易扩展

向 Redis Cluster 中添加新节点，或者移除节点，都是透明的，不需要停机。水平、垂直方向都非常容易扩展。数据分区，海量数据，数据存储

- 原生

部署 Redis Cluster 不需要其他的代理或者工具，而且 Redis Cluster 和单机 Redis 几乎完全兼容。

### **4.2 分区**

不同节点分组服务于相互无交集的分片（sharding），Redis Cluster 不存在单独的 proxy 或配置服务器，所以需要将客户端路由到目标的分片。

### **4.2.1 客户端路由**

Redis Cluster 的客户端相比单机 Redis 需要具备路由语义的识别能力，且具备一定的路由缓存能力。
**moved 重定向：**

1. 每个节点通过通信都会共享 Redis Cluster 中槽和集群中对应节点的关系
2. 客户端向 Redis Cluster 的任意节点发送命令，接收命令的节点会根据 CRC16 规则进行 hash 运算与 16384 取余，计算自己的槽和对应节点
3. 如果保存数据的槽被分配给当前节点，则去槽中执行命令，并把命令执行结果返回给客户端
4. 如果保存数据的槽不在当前节点的管理范围内，则向客户端返回 moved 重定向异常
5. 客户端接收到节点返回的结果，如果是 moved 异常，则从 moved 异常中获取目标节点的信息
6. 客户端向目标节点发送命令，获取命令执行结果

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669883648-b05f6728-6af0-4b13-801c-a6ff59a5e6e7.png#averageHue=%23fbfbf9&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ue4433df0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=374&originWidth=807&originalType=url∶=1&rotation=0&showTitle=false&size=189813&status=done&style=none&taskId=u1145bf73-8e02-4803-a9a7-50adee10a81&title=)
**ask 重定向：**
在对集群进行扩容和缩容时，需要对槽及槽中数据进行迁移，当客户端向某个节点发送命令，节点向客户端返回 moved 异常，告诉客户端数据对应的槽的节点信息如果此时正在进行集群扩展或者缩空操作，当客户端向正确的节点发送命令时，槽及槽中数据已经被迁移到别的节点了，就会返回 ask，这就是 ask 重定向机制 1.客户端向目标节点发送命令，目标节点中的槽已经迁移支别的节点上了，此时目标节点会返回 ask 转向给客户端 2.客户端向新的节点发送 Asking 命令给新的节点，然后再次向新节点发送命令 3.新节点执行命令，把命令执行结果返回给客户端
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669883714-5702e789-9ac0-4a10-b705-281467f3b588.png#averageHue=%23fbfbf9&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u69b2d9d5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=401&originWidth=779&originalType=url∶=1&rotation=0&showTitle=false&size=148107&status=done&style=none&taskId=u0625ebb8-e31a-4100-9fa5-6e593b3839e&title=)
moved 和 ask 的区别
1、moved：槽已确认转移
2、ask：槽还在转移过程中
**Smart 智能客户端：**
**JedisCluster**
JedisCluster 是 Jedis 根据 RedisCluster 的特性提供的集群智能客户端，JedisCluster 为每个节点创建连接池，并跟节点建立映射关系缓存（Cluster slots）。JedisCluster 将每个主节点负责的槽位一一与主节点连接池建立映射缓存，JedisCluster 启动时，已经知道 key,slot 和 node 之间的关系，可以找到目标节点。JedisCluster 对目标节点发送命令，目标节点直接响应给 JedisCluster。如果 JedisCluster 与目标节点连接出错，则 JedisCluster 会知道连接的节点是一个错误的节点，此时节点返回 moved 异常给 JedisCluster，JedisCluster 会重新初始化 slot 与 node 节点的缓存关系，然后向新的目标节点发送命令，目标命令执行，命令并向 JedisCluster 响应。如果命令发送次数超过 5 次，则抛出异常"Too many cluster redirection!"
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669883730-a04eaa14-9e1b-48eb-9b86-bd16adf9c128.png#averageHue=%23f9f9f7&clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u47ee36a8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=400&originWidth=826&originalType=url∶=1&rotation=0&showTitle=false&size=174952&status=done&style=none&taskId=uab0390c3-7ec4-4aa9-9bdd-4f4551ebdeb&title=)
JedisPoolConfig config **=** **new** JedisPoolConfig**();** Set**<**HostAndPort**>** jedisClusterNode **=** **new** HashSet**<**HostAndPort**>();** jedisClusterNode**.**add**(new** HostAndPort**(**"192.168.127.128"**,** 7001**));** jedisClusterNode**.**add**(new** HostAndPort**(**"192.168.127.128"**,** 7002**));** jedisClusterNode**.**add**(new** HostAndPort**(**"192.168.127.128"**,** 7003**));** jedisClusterNode**.**add**(new** HostAndPort**(**"192.168.127.128"**,** 7004**));** jedisClusterNode**.**add**(new** HostAndPort**(**"192.168.127.128"**,** 7005**));** jedisClusterNode**.**add**(new** HostAndPort**(**"192.168.127.128"**,** 7006**));** JedisCluster jcd **=** **new** JedisCluster**(**jedisClusterNode**,** config**);** jcd**.**set**(**"name:001"**,**"zhangfei"**);** String value **=** jcd**.**get**(**"name:001"**);**

### **4.2.2 迁移**

在 RedisCluster 中每个 slot 对应的节点在初始化后就是确定的。在某些情况下，节点和分片需要变更：

- 新的节点作为 master 加入；
- 某个节点分组需要下线；
- 负载不均衡需要调整 slot 分布。

此时需要进行分片的迁移，迁移的触发和过程控制由外部系统完成。包含下面 2 种：

- 节点迁移状态设置：迁移前标记源/目标节点。
- key 迁移的原子化命令：迁移的具体步骤。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669883908-76f19210-dc51-411e-9e7a-d8b49384a727.png#clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u102aa8d8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=266&originWidth=516&originalType=url∶=1&rotation=0&showTitle=false&size=104105&status=done&style=none&taskId=ufda99b54-5f48-4783-a0b8-ffd7cb32c4f&title=)

1. 向节点 B 发送状态变更命令，将 B 的对应 slot 状态置为 importing。
2. 向节点 A 发送状态变更命令，将 A 对应的 slot 状态置为 migrating。
3. 向 A 发送 migrate 命令，告知 A 将要迁移的 slot 对应的 key 迁移到 B。
4. 当所有 key 迁移完成后，cluster setslot 重新设置槽位。

### **4.3 容灾**

### **4.3.1 故障检查**

集群中的每个节点都会定期地（每秒）向集群中的其他节点发送 PING 消息，如果在一定时间内(cluster-node-timeout)，发送 ping 的节点 A 没有收到某节点 B 的 pong 回应，则 A 将 B 标识为 pfail。 A 在后续发送 ping 时，会带上 B 的 pfail 信息， 通知给其他节点。如果 B 被标记为 pfail 的个数大于集群主节点个数的一半（N/2 + 1）时，B 会被标记为 fail，A 向整个集群广播，该节点已经下线。其他节点收到广播，标记 B 为 fail。

### **4.3.2 从节点选举**

raft，每个从节点，都根据自己对 master 复制数据的 offffset，来设置一个选举时间，offffset 越大（复制数据越多）的从节点，选举时间越靠前，优先进行选举。slave 通过向其他 master 发送 FAILVOER_AUTH_REQUEST 消息发起竞选，master 收到后回复 FAILOVER_AUTH_ACK 消息告知是否同意。slave 发送 FAILOVER_AUTH_REQUEST 前会将 currentEpoch 自增，并将最新的 Epoch 带入到 FAILOVER_AUTH_REQUEST 消息中，如果自己未投过票，则回复同意，否则回复拒绝。所有的**Master**开始 slave 选举投票，给要进行选举的 slave 进行投票，如果大部分 master node（N/2 +1）都投票给了某个从节点，那么选举通过，那个从节点可以切换成 master。
RedisCluster 失效的判定：
1、集群中半数以上的主节点都宕机（无法投票）
2、宕机的主节点的从节点也宕机了（slot 槽分配不连续）

### **4.3.3 变更通知**

当 slave 收到过半的 master 同意时，会成为新的 master。此时会以最新的 Epoch 通过 PONG 消息广播自己成为 master，让 Cluster 的其他节点尽快的更新拓扑结构(node.conf)。

### **4.3.4 主从切换**

### **4.3.4.1 自动切换**

就是上面讲的从节点选举

### **4.3.4.2 手动切换**

人工故障切换是预期的操作，而非发生了真正的故障，目的是以一种安全的方式(数据无丢失)将当前 master 节点和其中一个 slave 节点(执行 cluster-failover 的节点)交换角色

1. 向从节点发送 cluster failover 命令（slaveof no one）
2. 从节点告知其主节点要进行手动切换（CLUSTERMSG_TYPE_MFSTART）
3. 主节点会阻塞所有客户端命令的执行（10s）
4. 从节点从主节点的 ping 包中获得主节点的复制偏移量
5. 从节点复制达到偏移量，发起选举、统计选票、赢得选举、升级为主节点并更新配置
6. 切换完成后，原主节点向所有客户端发送 moved 指令重定向到新的主节点

以上是在主节点在线情况下。如果主节点下线了，则采用 cluster failover force 或 cluster failover takeover 进行强制切换。

### **4.3.5 副本漂移**

我们知道在一主一从的情况下，如果主从同时挂了，那整个集群就挂了。为了避免这种情况我们可以做一主多从，但这样成本就增加了。Redis 提供了一种方法叫副本漂移，这种方法既能提高集群的可靠性又不用增加太多的从机
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652669883778-ce8a84f3-a462-4cd3-98ff-3ad0591c77e6.png#clientId=u9f818cbd-8ce0-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u09829517&margin=%5Bobject%20Object%5D&name=image.png&originHeight=395&originWidth=758&originalType=url∶=1&rotation=0&showTitle=false&size=100837&status=done&style=none&taskId=u90546b0e-8600-4103-b28f-e4ed9d1b2db&title=)
Master1 宕机，则 Slaver11 提升为新的 Master1，集群检测到新的 Master1 是单点的（无从机），集群从拥有最多的从机的节点组（Master3）中，选择节点名称字母顺序最小的从机（Slaver31）漂移到单点的主从节点组(Master1)。具体流程如下（以上图为例）：

1. 将 Slaver31 的从机记录从 Master3 中删除
2. 将 Slaver31 的的主机改为 Master1
3. 在 Master1 中添加 Slaver31 为从节点
4. 将 Slaver31 的复制源改为 Master1
5. 通过 ping 包将信息同步到集群的其他节点
