---
title: Redis 数据类型
urlname: nzrq33
date: '2022-05-07 12:07:43 +0800'
tags:
  - redis
categories:
  - Redis
---

### redis 简介

redis 是一个 K-V 的存储系统，使用 ANSI C 编写，key 是字符串型，value 的类型有 string、list、set、sortset、hash、bitmap、geo、stream，
redis 命令不区分大小写，k-v 区分大小写。

### 结构

#### redisDB 结构

```c
typedef struct redisDb {
	int id; //id是数据库序号，为0-15（默认Redis有16个数据库）
	long avg_ttl; //存储的数据库对象的平均ttl（time to live），用于统计
	dict *dict; //存储数据库所有的key-value
	dict *expires; //存储key的过期时间
	dict *blocking_keys;//blpop 存储阻塞key和客户端对象
	dict *ready_keys;//阻塞后push 响应阻塞客户端 存储阻塞后push的key和客户端对象
	dict *watched_keys;//存储watch监控的的key和客户端对象
} redisDb;
```

#### RedisObject 结构

```c
typedef struct redisObject {
	unsigned type:4;//类型 五种对象类型  REDIS_STRING(字符串)、REDIS_LIST (列表)、REDIS_HASH(哈希)、REDIS_SET(集合)、REDIS_ZSET(有序集合)。
	unsigned encoding:4;//编码 4表示位数
	void *ptr;//指向底层实现数据结构的指针,指向具体数据
	//...
	int refcount;//引用计数
	//...
	unsigned lru:LRU_BITS; //LRU_BITS为24bit 记录最后一次被命令程序访问的时间
        //高16位存储一个分钟数级别的时间戳，低8位存储访问计数（lfu ： 最近访问次数）
	//...
} robj;

## 获取对象类型与编码
type key 返回对象类型（String...）
object encoding key 获取编码
```

### redis 为什么快?

> redis 官网给出的速度为 10 万/s

1. redis 完全基于内存,所以读写效率高.redis 支持持久化操作,持久化操作都是 fork 子进程和利用 Linux 页缓存技术完成不会影响 redis 性能
2. 单线程操作,避免线程频繁切换带来的性能损耗,频繁的上下文切换会影响性能
3. 采用了非阻塞 I/O 多路复用机制,多路复用 I/O 是利用 select、poll、epoll 可以同事监察多个流的 I/O 事件的能力,在空闲时间会把当前线程阻塞掉,当一个或者多个流有 I/O 事件就会从阻塞中唤醒,于是程序就会轮询一遍所有的流,并且只依次顺序的处理就绪的流,这种做法避免了大量的无用操作

### redis 有哪些数据类型?

#### 基本类型

1. string 字符串
2. hash 哈希
3. list 列表
4. set 集合
5. zset 有序集合

#### 特殊数据类型

1. geospatial (鸡儿四白首)地理位置
2. hyperloglog（黑 perloglog） 基数估值算法一个集合中不重复的元素个数,提供了不精确的去重计数方案,场景:统计 UV
3. bitmap 位操作,场景:打卡,消息已读未读

### Redis 数据类型以及应用场景

#### String 类型（放置字符串，整数，浮点数）

常用命令：

```bash
set key vakue #赋值
get key #获取值
getset key value #取值
setnx key value #当 value 不存在时采用赋值【分布式锁】
set key value NX PX 3000 #原子操作，px 设置毫秒数
append key value #尾部追加值
strlen key #获取字符串长度
incr key #递增数字【乐观锁】
incrby key increment #增加指定整数
desc key #递减数字【乐观锁】
desc key increment #减少指定整数
```

对应 encoding：int（int 数据类型）、embstr（长度小于 44 个字节）raw（长度大于 44 个字节）

#### lis 列表类型（储存有序，可重复元素）

```bash
lpush key v1 v2 ... #从左侧插入列表
lpop key #从列表左侧取出
rpush key v1 v2 ... #从右侧插入列表
rpop key #从列表中取出
lpushx key value #将值插入列表头部
rpushx key value #将值插入列表尾部
blpop key timeout #从列表左侧取出，当列表为空时阻塞，可以设置最大阻塞时间，单位秒
llen key #获取列表元素个数
lindex key index #获取下标 index 元素
lrange key start end #返回列表指定区间元素，区间通过指定 start 和 end 指定
lrem key count value #删除列表中与 value 相等的元素
ltrim key start end #对列表进行修剪，保留 start 和 end 区间
rpoplpush key1 key2 #从key1列表右侧弹出并插入到 key2 列表左侧
brpoplpush key1 key2 #从key1列表右侧弹出并插入到key2列表左侧，会阻塞
linsert key BEFORE/AFTER pivot value #将value插入到列表，且位于值pivot之前或之后
```

对应 encoding：quicklist(快速列表)

#### Set 集合类型（无序，唯一元素）

```bash
sadd key member1 member2 member3... #向集合添加元素
srem key mem1 mem2 ... #删除集合指定元素
smembers key #获取集合中所有元素
spop key #返回集合中随机一个元素并将该元素删除
srandmenber key #返回集合中随机一个元素，不会删除元素
scard key #获取集合元素数量
sismember key member #判断元素是否在集合内
sinter key1 key2 #求两个集合交集
sdiff key1 key2 #求两个集合差集
sunion key1 key2 #求两个集合并集
```

encoding：

1. inset：元素都是整数并且处于 64 位有符号整数范围内。
2. dict：元素都是整数并且都处于 64 位有符号整数范围外。

#### sortedset 有序集合（分数排序，分数可重复）

```bash
zadd key score1 member1 #为有序集合添加新成员
zrem key mem1 mem2 #为有序集合删除指定成员
zcard key #获取有序集合的元素个数
zcount key min max #获取 score 值在指定区间内的元素个数
zincrby key increment member #在集合 member 分值上加 increment
zscore key member #获取集合 member 分支
zrank key member #获取 member 在集合中排名，分值从小到大
zrevrank key member #获取 member 在集合中排名，分值从大到小
zrange key start end #获取集合中指定区间成员，分数从小到大
zrevrange key start end #获取集合中指定区间成员，分数从大到小
```

对应 encoding：

1. ziplist：元素的个数比较少，且元素都是小整数或者短字符串时
2. skiplist+dict：元素的个数比较多或元素不是小整数或者短字符串时

#### hash 类型（散列表）

```bash
hset key field value #赋值，不区别新增和修改
hmset key field1 value1 #批量赋值
hsetnx key field value #赋值，如果 key 存在则不操作
hexists key field #查看某个 field 是否存在于某个 key 中
hget key field #获取一个字段的值
hmget key field1 field2 #获取多个字段的值
hgetall key #获取 key 所有元素
hdel key field1 field #删除指定字段
hincrby key field increment #指定字段增长 increment
hlen key #获取字段数量
```

encoding：

1. dict：元素个数比较多或元素是大整数或是大字符串
2. ziplist：元素个数比较少，且元素是小整数或是小字符串

#### bitmap 位图类型（签到，统计活跃，用户在线状态）

```bash
setbit key offset value #设置 key 在 offset 处的 bit 值（只能是 0 或 1）
getbit key offset #获取 key 在 offset 的 git 值
bitcount key #获取 key 的 bit 位为 1 的个数
gitpos key value #返回第一个被设为 bit 值得索引
bitop and[or/xor/not] destkey key [key …] #对多个key 进行逻辑运算后存入destkey中
```

#### geo 地理位置类型（记录地理位置，计算距离，查找附近的人，本质 key value）

```bash
geoadd 经度 纬度  成员名称 经度1 纬度1 成员名称 2 #添加地理坐标
geohash key member #返回标准 gehash 串
geopos key member1 member2 #返回成员经纬度
geodist key member1 meber2 #计算成员间距离
georadiusbymember key member [半径 radius 数字] [m|km|mi|ft（单位）] count 数量 [asc|desc]
```

#### steam 数据流类型

```bash
1xadd key id <*> field1 value1.... 将指定消息数据追加到指定队列(key)中，*表示最新生成的id（当前时间+序列号）
xread [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...] 从消息队列中读取，COUNT：读取条数,BLOCK：阻塞读（默认不阻塞）key：队列名称 id：消息id
xrange key start end [COUNT] 读取队列中给定ID范围的消息 COUNT：返回消息条数（消息id从小到大）
xrevrange key start end [COUNT] 读取队列中给定ID范围的消息 COUNT：返回消息条数（消息id从大到小）
xdel key id 删除队列的消息
xgroup create key groupname id 创建一个新的消费组
xgroup destory key groupname 删除指定消费组
xgroup delconsumer key groupname cname 删除指定消费组中的某个消费者
xgroup setid key id 修改指定消息的最大id
xreadgroup group groupname consumer COUNT streams key 从队列中的消费组中创建消费者并消费数据consumer不存在则创建）
```

对应 encoding:主要使用了 listpack（紧凑列表）和 Rax 树（基数树）

- listpack 表示一个字符串列表的序列化，listpack 可用于存储字符串或整数。用于存储 stream 的消息内容。
- Rax 是一个有序字典树 (基数树 Radix Tree)，按照 key 的字典序排列，支持快速地定位、插入和删除操作。用于存储消息队列，在 Stream 里面消息 ID 的前缀是时间戳 + 序号，这样的消息可以理解为时间序列消息。使用 Rax 结构 进行存储就可以快速地根据消息 ID 定位到具体的消息，然后继续遍历指定消息 之后的所有消息。

#### hyperloglog （统计 ip、uv、活跃用户数）

```bash
pfadd key value1 value2 #添加
pfcount key #统计元素个数
pfmerge destkey sourcekey1 sourcekey2 #合并到一个中
```

只能统计数量，没有办法知道具体内容是什么。
