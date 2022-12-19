---
title: Redis 持久化
urlname: kmufep
date: '2022-05-07 12:24:57 +0800'
tags:
  - redis
categories:
  - Redis
---

### 1 为什么要持久化

Redis 是内存数据库，宕机后数据会消失。Redis 重启后快速恢复数据，要提供持久化机制。Redis 有两种持久化方式：**RDB 和 AOF**，注意：Redis 持久化不保证数据的完整性。当 Redis 用作 DB 时，DB 数据要完整，所以一定要有一个完整的数据源（文件、mysql）在系统启动时，从这个完整的数据源中将数据 load 到 Redis 中，数据量较小，不易改变。比如：字典库。可以通过 info 查看持久化信息

### 2 RDB

RDB（Redis DataBase），是 redis 默认的存储方式，RDB 方式是通过快照（ snapshotting ）完成的。这一刻的数据，不关注过程。
**2.1 触发 RDB 快照的方式**

1. 符合自定义配置的快照规则（在 redis.conf 中配置）
   save "" # 不使用 RDB 存储 不能主从 save 900 1 # 表示 15 分钟（900 秒钟）内至少 1 个键被更改则进行快照。 save 300 10 # 表示 5 分钟（300 秒）内至少 10 个键被更改则进行快照。 save 60 10000 # 表示 1 分钟内至少 10000 个键被更改则进行快照。
2. 执行 save 或者 bgsave 命令（在客户端输入命令）
3. 执行 flushall 命令
4. 执行主从复制操作 (第一次)

### 2.2 RDB 执行流程

1. Redis 父进程首先判断：当前是否在执行 save，或 bgsave/bgrewriteaof（aof 文件重写命令）的子进程，如果在执行则 bgsave 命令直接返回。
2. 父进程执行 fork（调用 OS 函数复制主进程）操作创建子进程，这个过程中父进程是阻塞的，Redis 不能执行来自客户端的任何命令。
3. 父进程 fork 后，bgsave 命令返回”Background saving started”信息并不再阻塞父进程，并可以响应其他命令。
4. 子进程创建 RDB 文件，根据父进程内存快照生成临时快照文件，完成后对原有文件进行原子替换。（RDB 始终完整）
5. 子进程发送信号给父进程表示完成，父进程更新统计信息。
6. 父进程 fork 子进程后，继续工作。
   **2.3 RDB 文件结构**
   1、头部 5 字节固定为“REDIS”字符串
   2、4 字节“RDB”版本号（不是 Redis 版本号），当前为 9，填充后为 0009
   3、辅助字段，以 key-value 的形式（redis-ver 版本，aof-preamble 是否开启 aof，redis-bits 位数，repl-stream-db 主从赋值，ctime 当前时间戳，repl-id 主从复制 id，used-mem 使用内存，repl-offset 主从复制）
   4、存储数据库号码
   5、字典大小
   6、过期 key
   7、主要数据，以 key-value 的形式存储
   8、结束标志
   9、校验和，就是看文件是否损坏，或者是否被修改。
   **2.4 RDB 优缺点**
   优点：RDB 是二进制压缩文件，占用空间小，便于传输（传给 slaver）主进程 fork 子进程，可以最大化 Redis 性能，主进程不能太大，复制过程中主进程阻塞
   缺点：不保证数据完整性，会丢失最后一次快照以后更改的所有数据

### 3 AOF

**3.1 AOF 概念**
AOF（append only fifile）是 Redis 的另一种持久化方式。Redis 默认情况下是不开启的。开启 AOF 持久化后，Redis 将所有对数据库进行过写入的命令（及其参数）（RESP）记录到 AOF 文件， 以此达到记录数据库状态的目的，，这样当 Redis 重启后只要按顺序回放这些命令就会恢复到原始状态了。AOF 会记录过程，RDB 只管结果
**3.2 AOF 实现**
配置 redis.conf

# 可以通过修改 redis.conf 配置文件中的 appendonly 参数开启 appendonly yes # AOF 文件的保存位置和 RDB 文件的位置相同，都是通过 dir 参数设置的。 dir ./ # 默认的文件名是 appendonly.aof，可以通过 appendfilename 参数修改 appendfilename appendonly.aof

**3.3 AOF 原理**
AOF 文件中存储的是 redis 的命令，同步命令到 AOF 文件的整个过程可以分为三个阶段：
命令传播：Redis 将执行完的命令、命令的参数、命令的参数个数等信息发送到 AOF 程序中。
缓存追加：AOF 程序根据接收到的命令数据，将命令转换为网络通讯协议的格式，然后将协议内容追加到服务器的 AOF 缓存中。
文件写入和保存：AOF 缓存中的内容被写入到 AOF 文件末尾，如果设定的 AOF 保存条件被满足的话，fsync 函数或者 fdatasync 函数会被调用，将写入的内容真正地保存到磁盘中。

1. 命令传播

当一个 Redis 客户端需要执行命令时， 它通过网络连接， 将协议文本发送给 Redis 服务器。服务器在接到客户端的请求之后， 它会根据协议文本的内容， 选择适当的命令函数， 并将各个参数从字符串文本转换为 Redis 字符串对象（ StringObject ）。每当命令函数成功执行之后， 命令参数都会被传播到 AOF 程序。 2. 缓存追加
当命令被传播到 AOF 程序之后， 程序会根据命令以及命令的参数， 将命令从字符串对象转换回原来的协议文本。协议文本生成之后， 它会被追加到 redis.h/redisServer 结构的 aof_buf 末尾。redisServer 结构维持着 Redis 服务器的状态， aof_buf 域则保存着所有等待写入到 AOF 文件的协议文本（RESP）。 3. 文件写入和保存
每当服务器常规任务函数被执行、 或者事件处理器被执行时， aof.c/flflushAppendOnlyFile 函数都会被调用， 这个函数执行以下两个工作：WRITE：根据条件，将 aof_buf 中的缓存写入到 AOF 文件。SAVE：根据条件，调用 fsync 或 fdatasync 函数，将 AOF 文件保存到磁盘中。
**3.4 AOF 保存模式**
Redis 目前支持三种 AOF 保存模式，它们分别是：
AOF_FSYNC_NO ：不保存。 AOF_FSYNC_EVERYSEC ：每一秒钟保存一次。（默认） AOF_FSYNC_ALWAYS ：每执行一个命令保存一次。（不推荐）
**不保存**
在这种模式下， 每次调用 flflushAppendOnlyFile 函数， WRITE 都会被执行， 但 SAVE 会被略过。SAVE 只会在以下任意一种情况中被执行：Redis 被关闭，AOF 功能被关闭，系统的写缓存被刷新（可能是缓存已经被写满，或者定期保存操作被执行）这三种情况下的 SAVE 操作都会引起 Redis 主进程阻塞。
**每一秒钟保存一次(默认)**
在这种模式中， SAVE 原则上每隔一秒钟就会执行一次， 因为 SAVE 操作是由后台子线程（fork）调用的， 所以它不会引起服务器主进程阻塞。
**每执行一个命令保存一次(不推荐)**
在这种模式下，每次执行完一个命令之后， WRITE 和 SAVE 都会被执行。另外，因为 SAVE 是由 Redis 主进程执行的，所以在 SAVE 执行期间，主进程会被阻塞，不能接受命令请求。AOF 保存模式对性能和安全性的影响

| **模式**           | **WRITE 是否阻塞** | **SAVE 是否阻塞** | **停机时丢失的数据量**                              |
| ------------------ | ------------------ | ----------------- | --------------------------------------------------- |
| AOF_FSYNC_NO       | 阻塞               | 阻塞              | 操作系统最后一次对 AOF 文件触发 SAVE 操作之后的数据 |
| AOF_FSYNC_EVERYSEC | 阻塞               | 不阻塞            | 一般情况下不超过 2 秒钟的数据                       |
| AOF_FSYNC_ALWAYS   | 阻塞               | 阻塞              | 最多只丢失一个命令的数据                            |

**3.5 AOF 重写**
AOF 记录数据变化越来越大，需要重写瘦身
Redis 可以在 AOF 体积变得过大时，自动地在后台（Fork 子进程）对 AOF 进行重写。重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 所谓的“重写”其实是一个有歧义的词语， 实际上，AOF 重写**并不需要对原有的 AOF 文件进行任何写入和读取**， 它针对的是数据库中键的当前值。
Redis 不希望 AOF 重写造成服务器无法处理请求， 所以 Redis 决定将 AOF 重写程序放到（后台）子进程里执行， 这样处理的最大好处是：

1. 子进程进行 AOF 重写期间，主进程可以继续处理命令请求。
2. 子进程带有主进程的数据副本，使用子进程而不是线程，可以在避免锁的情况下，保证数据的安全性。

因为子进程在进行 AOF 重写期间， 主进程还需要继续处理命令， 而新的命令可能对现有的数据进行修改， 这会让当前数据库的数据和重写后的 AOF 文件中的数据不一致。为了解决这个问题， Redis 增加了一个 AOF 重写缓存， 这个缓存在 fork 出子进程之后开始启用，Redis 主进程在接到新的写命令之后， 除了会将这个写命令的协议内容追加到现有的 AOF 文件之外，还会追加到这个缓存中。
**过程分析**
Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
当子进程在执行 AOF 重写时， 主进程需要执行以下三个工作：

1. 处理命令请求。
2. 将写命令追加到现有的 AOF 文件中。
3. 将写命令追加到 AOF 重写缓存中。

现有的 AOF 功能会继续执行，即使在 AOF 重写期间发生停机，也不会有任何数据丢失。所有对数据库进行修改的命令都会被记录到 AOF 重写缓存中。当子进程完成 AOF 重写之后， 它会向父进程发送一个完成信号， 父进程在接到完成信号之后， 会调用一个信号处理函数， 并完成以下工作：

1. 将 AOF 重写缓存中的内容全部写入到新 AOF 文件中。
2. 对新的 AOF 文件进行改名，**覆盖**原有的 AOF 文件。
3. Redis 数据库里的+AOF 重写过程中的命令------->新的 AOF 文件---->覆盖老的

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652064425954-0603812d-4a8f-4d1a-ba6f-c3f5761c0338.png#averageHue=%23f3f3f3&clientId=u5f187201-dfb7-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uce2fdc38&margin=%5Bobject%20Object%5D&name=image.png&originHeight=358&originWidth=1003&originalType=url∶=1&rotation=0&showTitle=false&size=140259&status=done&style=none&taskId=ub708b5ea-126e-41ba-8b49-8290c22c003&title=)

**3.6 AOF 触发条件**
1 配置触发

# 表示当前 aof 文件大小超过上一次 aof 文件大小的百分之多少的时候会进行重写。如果之前没有重写过，以 启动时 aof 文件大小为准 auto-aof-rewrite-percentage 100 # 限制允许重写最小 aof 文件大小，也就是文件大小小于 64mb 的时候，不需要进行优化 auto-aof-rewrite-min-size 64mb

2 命令触发
127.0.0.1:6379> bgrewriteaof Background append only file rewriting started
3 混合持久化
RDB 和 AOF 各有优缺点，Redis 4.0 开始支持 rdb 和 aof 的混合持久化。如果把混合持久化打开，aof rewrite 的时候就直接把 rdb 的内容写到 aof 文件开头。RDB 的头+AOF 的身体---->appendonly.aof 开启混合持久化
aof-use-rdb-preamble yes
在加载时，首先会识别 AOF 文件是否以 REDIS 字符串开头，如果是就按 RDB 格式加载，加载完 RDB 后继续按 AOF 格式加载剩余部分。
**3.7 AOF 文件的载入与数据还原**
因为 AOF 文件里面包含了重建数据库状态所需的所有写命令，所以服务器只要读入并重新执行一遍 AOF 文件里面保存的写命令，就可以还原服务器关闭之前的数据库状态 Redis 读取 AOF 文件并还原数据库状态的详细步骤如下：
1、创建一个不带网络连接的伪客户端（fake client）：因为 Redis 的命令只能在客户端上下文中执行，而载入 AOF 文件时所使用的命令直接来源于 AOF 文件而不是网络连接，所以服 务器使用了一个没有网络连接的伪客户端来执行 AOF 文件保存的写命令，伪客户端执行命令 的效果和带网络连接的客户端执行命令的效果完全一样
2、从 AOF 文件中分析并读取出一条写命令
3、使用伪客户端执行被读出的写命令
4、一直执行步骤 2 和步骤 3，直到 AOF 文件中的所有写命令都被处理完毕为止当完成以上步骤之后，AOF 文件所保存的数据库状态就会被完整地还原出来，整个过程如下图所示：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1652064425642-4ade1d65-7d1f-427c-9151-b96624d363fb.png#averageHue=%23f8f8f8&clientId=u5f187201-dfb7-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u9b920ffd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=592&originWidth=948&originalType=url∶=1&rotation=0&showTitle=false&size=89336&status=done&style=none&taskId=u6dcd504e-73f3-4fd6-b6f2-48d30c54e68&title=)

### 4 RDB 与 AOF 对比

1、RDB 存某个时刻的数据快照，采用二进制压缩存储，AOF 存操作命令，采用文本存储(混合)
2、RDB 性能高、AOF 性能较低
3、RDB 在配置触发状态会丢失最后一次快照以后更改的所有数据，AOF 设置为每秒保存一次，则最多丢 2 秒的数据
4、Redis 以主服务器模式运行，RDB 不会保存过期键值对数据，Redis 以从服务器模式运行，RDB 会保存过期键值对，当主服务器向从服务器同步时，再清空过期键值对。AOF 写入文件时，对过期的 key 会追加一条 del 命令，当执行 AOF 重写时，会忽略过期 key 和 del 命令
应用场景：
内存数据库 rdb+aof 数据不容易丢 ，缓存服务器 rdb 性能高 。不建议只使用 aof (性能差) ，在数据还原时有 rdb+aof 则还原 aof，因为 RDB 会造成文件的丢失，AOF 相对数据要完整。只有 rdb，则还原 rdb。
