---
title: redis8 类型
urlname: ighvtz
date: '2022-05-06 16:09:40 +0800'
tags:
  - redis
categories:
  - Redis
---

## redis 类型

1. Strings
2. Lists
3. Sets
4. Hashes
5. Sorted sets
6. Streams
7. Geospecial(地理位置)
8. HyperLogLog（基数统计）
9. Bitmaps（位图）
10. Bitfields（比特字段）

redis 如果 key 的值为空时，redis 会自动销毁该键，无需提前定义该键的值类型，但是如果设置了该值，却用了其他类型的操作去操作该键会报异常。

### String

支持保存文本、序列化对象、二进制数组，允许进行计数器、位操作。

```bash
#设置值
set mykey somevalue
#获取值
get mykey
#检测如果存在就不设置
set mykey newval nx
#强制设置
set mykey newval xx
#设置时指定过期时间
set request 1 ex 30 nx
#自增/自减 (incr/decr) 该操纵是原子的不会出现竞争情况，因为 redis 从设计上是单进程单线程
set counter 100
incr counter
incrby counter 10 #指定增加步长
#为了减少延迟，可以批量设置值以及读取
mset a 10 b 20 c 30
mget a b c

```

有些命令不是针对特定类型的才能使用，他能与任何类型的键来配合使用

```bash
 set mykey hello
#检测键是否存在
exists mykey
#删除键
del mykey
#获取键类型
type mykey
#设置键值并加上过期时间
set key 100 ex 10
#查询过期时间
ttl key
```

过期时间：
过期时间的粒度为毫秒级，过期的时间会被复制保存到磁盘上，当服务器宕机
锁续命：

-

锁 key 丢失：

- 写脚本恢复
- Redlock

### Lists

> redis 中的 list 是通过双向循环链表（quickList）实现的，为什么链表以及这样做的好处是，
>
> 1. 双端链表，获取前后节点复杂度为 O(1)
> 2. 无环，对链表的访问以 null 为终点
> 3. 带头和尾节点，获取头尾节点的时间复杂度为 O(1)
> 4. 链表计数器，对访问链表大小的时间复杂度为 O(1)

Redis 早期版本存储 list 列表数据结构使用的是压缩列表 ziplist 和普通的双向链表 linkedlist，也就是元素少时用 ziplist，元素多时用 linkedlist。后续版本对列表数据结构进行了改造，使用 quicklist 代替了 ziplist 和 linkedlist。提高了新增和删除的效率,但是遍历效率相对较低

> 数组和链表区别：
> 结构上:
>
> - 数组必须先定义好固定的长度（元素个数），不能适应数据的动态增减功能。
> - 链表动态的进行存储分配，可以适应数据动态增减情况，方便插入、删除数据。
>
> 内存存储：
>
> - （静态从栈中分配空间），快速但自由度小。
> - 数组在内存中是按顺序存储的，链表是随机的。
> - 数组按下标索引访问，速度比较快，但插入元素的话就得移动很多元素，所以数组插入效率低。
> - 链表从堆中分配空间，自由但申请麻烦
> - 链表获取长度比较快

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1668650580784-592a03db-aaed-4591-b3d6-f6e0239148a6.png#averageHue=%23f4f4ea&clientId=u5b5872ff-fa81-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=209&id=u2f28c36c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=417&originWidth=1030&originalType=binary∶=1&rotation=0&showTitle=false&size=172081&status=done&style=none&taskId=u2c56e547-0398-4d6f-afad-1d61554e11c&title=&width=515)

```bash
rpush mylist A
lrange mylist 0 -1
#多个变量
rpush mylist 1 2 3 4 5 "foo bar"
rpop mylist
#使用ltrim保持指定范围，丢弃掉后边的内容
ltrim mylist 0 2
#传统的使用队列，会出现 CPU 空转问题，redis 使用阻塞方式解决这个问题
#使用阻塞到达指定时间没有获取到值，返回nil
brpop tasks 5
brpop tasks 0 #如果设为 0 就是永远等待元素直至出现
```

应用：

- 如发布我的动态。
- 消息队列

### Hashes

> [散列表](https://baike.baidu.com/item/%E6%95%A3%E5%88%97%E8%A1%A8/10027933?fromModule=lemma_inlink)（Hash table，也叫哈希表），是根据关键码值(Key value)而直接进行访问的[数据结构](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/1450?fromModule=lemma_inlink)。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做[散列函数](https://baike.baidu.com/item/%E6%95%A3%E5%88%97%E5%87%BD%E6%95%B0/2366288?fromModule=lemma_inlink)，存放记录的[数组](https://baike.baidu.com/item/%E6%95%B0%E7%BB%84/3794097?fromModule=lemma_inlink)叫做[散列表](https://baike.baidu.com/item/%E6%95%A3%E5%88%97%E8%A1%A8/10027933?fromModule=lemma_inlink)。

[hash](https://so.csdn.net/so/search?q=hash&spm=1001.2101.3001.7020)类型是一个 string 类型的 field 和 value 的映射表，每个 hash 可以存储 232 - 1 键值对（40 多亿），hash 对存放的 key 和 val 数量没有限制

```bash
hset user:1000 username antirez birthyear 1977 verified 1
#获取
hget user:1000 username
hgetall user:1000
#获取多个字段
hmget user:1000 username birthyear no-such-field
#使用 hincrby 可以在 hash 中递增
hincrby user:1000 birthyear 10
```

应用：

- 购物车（以用户 id 为 key，商品 id 为 field，商品数量为 value，恰好构成了购物车的 3 个要素）![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1668655469782-1933c4f4-dcb7-47e5-8ee8-dae48fe7680f.png#averageHue=%23efeeee&clientId=u5b5872ff-fa81-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=288&id=u7a1276a3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1266&originWidth=1122&originalType=binary∶=1&rotation=0&showTitle=false&size=331157&status=done&style=none&taskId=u741b52b4-9a5b-4ede-a4d9-db2b478201c&title=&width=255)
- 存储对象

### Sets

redis set 是无序结合，可以执行交集、并集、差集计算

```bash
#向集合添加元素
sadd myset 1 2 3
#查看集合成员
smembers myset
#检查元素是否存在集合
sismember myset 3

sadd news:1000:tags 1 2 5 77
#计算交集
sinter tag:1:news tag:2:news
#将一个集合复制到另一个集合
sunionstore game:1:deck deck
#随机剔除一个值
spop game:1:deck

#获取随机元素不从集合中删除
SRANDMEMBER deck 4
```

使用场景：

- 给文章设置标签
- 好友关系
- 限流

### Sorted sets

> sorted set 是 hash 和 set 的混合，如果两个元素 score 相同，那么根据字符排序。分说是从大到小排序。

**如何实现的？**
实现注意:排序集是通过一个双端口数据结构实现的，它包含一个跳跃列表和一个哈希表，所以每次我们添加一个元素时，Redis 都会执行一个 O(log(N))操作。这很好，但是当我们请求排序的元素时，Redis 根本不需要做任何工作，它已经全部排序了:

跳跃表：增加了向前指针的链表叫作跳表。跳表全称叫做跳跃表，简称跳表。跳表是一个随机化的数据结构，实质就是一种可以进行二分查找的有序链表。跳表在原有的有序链表上面增加了多级索引，通过索引来实现快速查找。跳表不仅能提高搜索性能，同时也可以提高插入和删除操作的性能。。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1668733613457-cbd262a7-e7d8-4725-bebd-7644d491655a.png#averageHue=%23f6f5f5&clientId=u3fa3eca6-4a5d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=171&id=u96f05e51&margin=%5Bobject%20Object%5D&name=image.png&originHeight=342&originWidth=1161&originalType=binary∶=1&rotation=0&showTitle=false&size=25936&status=done&style=none&taskId=u39d23fe6-f5f6-4f46-b342-d7f927b699c&title=&width=580.5)

```bash
#设置值
zadd hackers 1940 '张三'
#获取排名
zrange hackers 0 -1
#按照相反的顺序排列
zrevrange hackers 0 -1
#携带分数返回
zrevrange hackers 0 -1 withscores
#通过 score 来过滤
zrangebyscore hackers min max
zrangebyscore hackers -inf 1950
#通过 score 删除元素
zremrangebyscore hackers 1940 1960
#获取 rank 排名
zrank hackers '王五'
#反序获取 rank 排名
zrevrank hackers '王五'

zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0 "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"0 "Linus Torvalds" 0 "Alan Turing"
#在 2.8 版本以后可以通过字典顺序获取范围
zrangebylex hackers [B [P
#统计数量
zcard hackers
```

补充说明
排序集的分数可以随时更新。只需对已包含在排序集中的元素调用 ZADD，就会以 O(log(N))时间复杂度更新其分数(和位置)。因此，排序集适用于有大量更新的情况。 由于这个特点，一个常见的用例就是排行榜。典型的应用程序是 Facebook 游戏，在该游戏中，您结合了根据用户的高分进行排序的功能，以及获取排名操作，以便显示排名前 n 的用户和排名后的用户

### Bitmaps

> 位图不是实际的数据类型，而是在 String 类型上定义的一组面向位的操作。因为字符串是二进制安全 blob，它们的最大长度是 512 MB，所以它们适合设置 2^32 个不同的位。 位操作分为两组:固定时间的单位操作，如将位设置为 1 或 0，或获取其值，以及对位组的操作，如计算给定位范围内的集合位的数量(例如，总体计数)。 位图的最大优点之一是，在存储信息时，它们通常可以节省极大的空间

```bash
setbit key 10 1 #setbit key offset value
getbit key 10



```

SETBIT 命令的第一个参数是位号，第二个参数是要设置位的值，即 1 或 0。如果寻址位在当前字符串长度之外，该命令将自动放大字符串。
GETBIT 只返回指定索引处的位的值。超出范围的位(寻址存储在目标键中的字符串长度以外的位)总是被认为是零。

- bittop 提供按位操作，and、or、xor、not
- bitcount 整体统计
- bitpos 获取指定 position 的 value

**应用：**

- **实时统计（如签到）**
- 存储空间高效但高性能的布尔值信息与对象 id 相关联。

### HyperLogLog

> HyperLogLog 是一种概率数据结构，用于计数唯一的东西(技术上这指的是估计集合的基数)。通常计数唯一项需要使用与您想要计数的项数量成比例的内存，因为您需要记住过去已经见过的元素，以避免多次计数。然而，有一组算法是用内存来换取精度的:你以一个标准误差的估计度量结束，在 Redis 实现的情况下，这个标准误差小于 1%。
> hyperLoglog 不会存储每个元素的值，通过存储 hash 元素的一个 1 的位置，来计算数量。
> HyperLogLog 的优点在于，输入元素的数量或者体积非常大时，基数计算的存储空间是固定的
> HyperLogLog 的算法设计能使用 12k 的内存来近似的统计 2^64 个数据

对比 set 做 uv

- 如果 uv 上限很高，那么 set 的空间开销就很大。

```bash
pfadd uv '127.0.0.1' '127.0.0.1' '192.168.1.1'
pfcount uv
#合并
pfmerge uv1 uv2
```

### Stream

> Redis 流是一种类似于仅追加日志的数据结构。您可以使用流实时记录并同时联合事件。
> Redis 流用例的例子包括: 事件来源(例如，跟踪用户操作、点击等) 传感器监测(例如，现场设备的读数)
> 通知(例如，在单独的流中存储每个用户的通知记录) Redis 为每个流条目生成一个唯一的 ID。您可以使用这些 id 稍后检索它们相关联的条目，或者读取和处理流中的所有后续条目。

```bash
#新增
xadd event:user:20221020 *  view_page 10s
#再次新增（链式结构）
xadd event:user:20221020 *  click-ad banner
#获取长度
xlen event:user:20221020
#裁剪
xtrim event:user:20201020 maxlen 1
#获取条目   - 的含义表示一个最小的id, + 号则表示一个最大的id
xrange event:user:20221020 - +
#如果特别多使用 count 约束
xrange event:user:20221020 - + count 1
#反序查找
xrevrange event:user:20221020 + -
#只查询最新消息
#XREAD 用来监听到达 stream 的消息.
#count 是一个非强制选项,用来约束一次读取的条目数目.
#streams 是一个强制选项,用来指定读取的 stream . 0 表示只获取比指定 id 0 大的条目.
xread count 1 streams event:user:20221020 0
```

## RDB 和 AOF

RDB 和 AOF
RDB（reids database）
AOF（append on file）

RDB：指定时间间隔内将内存数据集快照写入磁盘中，实际操作就是 fork 一个子进程，将数据集写入临时文件，写入成功后再替换原来的文件，用二进制压缩存储。
**优点：**

1. 整个 redis 数据库只包含一个 dump.rdb,方便持久化
2. 容灾性好，方便备份
3. 性能最大化，fork 子进程来完成写操作，让主进程继续处理命令，所以 IO 最大化。使用单独子进程进行持久化，主进程不会进行任何 IO 操作，保证 redis 高性能。
4. 数据集大时比 AOF 启动效率更高。

**缺点：**

- 数据安全性低。
- 由于 RDB 是通过 fork 子进程来协助完成数据持久化工作的，因此当数据集非常大时，可能导致服务停止几百毫秒，甚至 1s

AOF：
以日志形式记录服务器每一个写、删除操作，以文本方式记录，可以打开文件查看详细的操作记录。
**优点：**

- **数据安全，redis 提供了三种同步策略：1）每秒同步 2）每修改同步 3）不同步**
- **容易解决数据一致性问题，可以通过 redis-check-aof 工具解决数据一致性问题**
- **节省空间，写模式定期对 AOF 文件进行重写，已达到压缩目的**

**缺点：**

- AOF 比 RDB 文件大，且恢复速度慢
- 数据集大时比 RDB 启动效率低
- 运行效率没有 RDB 高

## Redis 单线程为什么这么快

原因：

- 首先 redis 是基于内存的，所以操作非常快，多线程会出现线程切换问题，多线程切换会出现上下文切换，切换时会消耗 CPU
- 多路 IO 复用，reids 基于 Reactor 模式开发了网络时间处理器，文件事件处理器 file event handle，他是单线程的所以 redis 才叫单线程，一个线程同时监听多个 socket，文件事务处理器包含（多个 socket 、IO 复用程序、文件事件分派器、事件处理器）
