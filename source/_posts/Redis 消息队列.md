---
title: Redis 消息队列
urlname: kuus56yzanumypi5
date: '2022-11-21 11:44:57 +0800'
tags:
  - redis
categories:
  - Redis
---

## List 实现消息队列

> 最简单的消息队列实现

缺点：

- 没有 ack 机制，需要手动实现 ack （Acknowledge）机制。需要新建一个队列当做备份，处理完删除。
- 无法持久化。
- 需要使用 while 循环的处理逻辑，性能上有损失，
- 同一消息不会被多个消费者消费。

```bash
#lpush、rpop 左进右出
#rpush、lpop 右进左出
#lpush、brpop 左进右出，阻塞
#rpush、blpop 右进左出，阻塞
#zset 可以做延时队列，将时间戳设为 score ，超时了就删除
```

## 发布/订阅

![](https://cdn.nlark.com/yuque/0/2022/webp/25799318/1669003061456-8ab13b01-2fc1-4bd2-a922-4df9dcdbb597.webp#averageHue=%23faf7f6&clientId=u42e4b57a-d677-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u3d2a9b9d&margin=%5Bobject%20Object%5D&originHeight=439&originWidth=1080&originalType=url∶=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud65c5b3b-820b-4b1a-bda7-5d00b9b5119&title=)
缺点：

- 无法持久化。
- 没有 ack 机制保证数据可靠性。

```bash
#订阅频道
subscribe tasks;
#匹配订阅频道
psubscribe tasks.*;
#发布消息
publish tasks.order 'hello world';

#退订消息
unsubscribe task;
#退订匹配频道
punsubscribe task.*;
```

## Streams 实现消息队列

优点：

- 可持久化
- 有 ack 机制
- 数据结构是链表实现

![](https://cdn.nlark.com/yuque/0/2022/webp/25799318/1669004465392-3c4024f4-a096-43e3-9aae-88c754166709.webp#averageHue=%23fdfdfb&clientId=u42e4b57a-d677-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u4eca8fc7&margin=%5Bobject%20Object%5D&originHeight=478&originWidth=1080&originalType=url∶=1&rotation=0&showTitle=false&status=done&style=none&taskId=ube4e5040-14e5-44c8-8b82-078125d9378&title=)

```bash
#添加
xadd task * order_id 20221125;
#查看消息列表
xrange task - +;
#获取长度
xlen tasks;
#修剪限制长度
xtrim tasks MAXLEN 2;
#以非阻塞方式获取列表
xread count 1 streams tasks 1
#以阻塞方式获取列表
xread count 1 block 1000 streams tasks 1

#创建消费者
#创建消费者组的时候必须指定 ID, ID 为 0 表示从头开始消费，为 $ 表示只消费新的消息，
xgroup create tasks tasks_group 0;
#消费者消费 >表示所有
xreadgroup group tasks_group client1 count 1 streams tasks >
#查看消费组信息
xinfo groups tasks

1) 1) "name"
   2) "tasks_group" #消费组名字
   3) "consumers"
   4) (integer) 1 #消费者数
   5) "pending"
   6) (integer) 4 #还没有 ack 的数量
   7) "last-delivered-id"
   8) "1669005783591-0"

#ack掉指定消息

```

Redis Stream 借鉴了很多 Kafka 的设计。

- **Consumer Group**：有了消费组的概念，每个消费组状态独立，互不影响，一个消费组可以有多个消费者
- **last_delivered_id** ：每个消费组会有个游标 last_delivered_id 在数组之上往前移动，表示当前消费组已经消费到哪条消息了
- **pending_ids** ：消费者的状态变量，作用是维护消费者的未确认的 id。pending_ids 记录了当前已经被客户端读取的消息，但是还没有 ack。如果客户端没有 ack，这个变量里面的消息 ID 会越来越多，一旦某个消息被 ack，它就开始减少。这个 pending_ids 变量在 Redis 官方被称之为 PEL，也就是 Pending Entries List，这是一个很核心的数据结构，它用来确保客户端至少消费了消息一次，而不会在网络传输的中途丢失了没处理。

![](https://cdn.nlark.com/yuque/0/2022/webp/25799318/1669005841701-4cdf0efc-6f18-4a7b-85f2-02fa4640b97a.webp#averageHue=%23fcfefb&clientId=u42e4b57a-d677-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=349&id=ubebed944&margin=%5Bobject%20Object%5D&originHeight=634&originWidth=1080&originalType=url∶=1&rotation=0&showTitle=false&status=done&style=none&taskId=u62b4f32b-6901-49b5-968f-37b6a5fdf1d&title=&width=594)
Stream 不像 Kafak 那样有分区的概念，如果想实现类似分区的功能，就要在客户端使用一定的策略将消息写到不同的 Stream。

- xgroup create：创建消费者组
- xgreadgroup：读取消费组中的消息
- xack：ack 掉指定消息

![](https://cdn.nlark.com/yuque/0/2022/webp/25799318/1669005869831-3eb8fb04-002d-45d3-aaac-3e23012be526.webp#averageHue=%23fcfcfa&clientId=u42e4b57a-d677-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u5a88fdeb&margin=%5Bobject%20Object%5D&originHeight=292&originWidth=1080&originalType=url∶=1&rotation=0&showTitle=false&status=done&style=none&taskId=u683ddbc9-1b06-4457-9788-60faccd2b9b&title=)
