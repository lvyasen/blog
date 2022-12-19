---
title: Redis 扩展功能
urlname: lvf887
date: '2022-05-09 10:03:04 +0800'
tags:
  - redis
categories:
  - Redis
---

### 发布订阅

reids 提供发布订阅功能，可用于消息传输。
redis 发布订阅包括三个部分

1. publisher （客户端）
2. subscriber （客户端）
3. channel （服务器端）

频道订阅退订命令

```markdown
# 订阅 subscriber

subscribe channel1 channel2

# 发布消息 publisher

publish channel message
```

### 事务

指单个逻辑工作单元执行的一系列操作

#### redis 事务

- 通过 multi、exec、discard、watch 四个命令完成
- 单个命令都是原子性的，所以这里要确保事务性的对象是命令集合
- 不支持回滚
- redis 将命令集合序列化并确保同一事务的命令集合连续且不被打断

#### 事务命令

1. multi：用于标记事务开始，redis 会将后续的命令逐个放入队列
2. exec：原子化执行命令队列。
3. discard：清除命令队列。
4. watch：监视 key

#### 弱事务性

- 语法错误：整个事务的命令队列都清除
- 运行错误：在队列正确的命令可以执行

reids 为什么不支持事务回滚？

1. 大多数错误是因为**语法错误**或**类型错误**，这两种错误在开发阶段是可以预见的。
2. 为了性能 忽略了事务回滚。

### 慢查询日志

在 redis.conf 中可以配置

```markdown
#执行时间超过多少微秒的命令请求会被记录到日志上
slowlog-log-slower-than 1000 #存储慢查询日志条数
slowlog-max-len 128
```

使用 config set 方式可以临时设置，重启后无效。
