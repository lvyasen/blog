---
title: Reids 通讯协议
urlname: nibkme
date: '2022-05-08 08:17:26 +0800'
tags:
  - redis
categories:
  - Redis
---

> redis 是单进程、单线程。应用系统和 redis 通过 redis 协议（RESP）进行交互

### 请求响应模式

> redis 协议位于 TCP 层之上，即客户端和 redis 实例保持双工的连接

#### 串行的请求响应模式（ping-pong）

> 串行化是最简单模式，客户端与服务端建立长连接，连接通过心跳机制检测（ping-pong）ack 应答，

流程：客户端发送请求，服务端响应，客户端收到响应，在发起第二次请求，服务端再响应。
telnet 和 redis-cli 发出的命令都属于该模式。
特点：**有问有答，耗时在网络传输命令，性能较低**

#### 双工请求响应（pipeline）

> 批量请求，批量响应。请求响应交叉进行，不会混淆（TCP 双工）

- pipline 的作用是将一批命令进行打包，然后发送给服务器，服务器执行完成后按顺序打包返回。
- 通过 pipline，一次 pipeline（n 条命令）=一次网络时间 + n 次命令时间

```bash
Jedis redis = new Jedis("192.168.1.111", 6379);
 redis.auth("12345678");//授权密码 对应redis.conf的requirepass密码
Pipeline pipe = jedis.pipelined();
for (int i = 0; i <50000; i++) {
pipe.set("key_"+String.valueOf(i),String.valueOf(i));
}
//将封装后的PIPE一次性发给redis
 pipe.sync();

```

#### 原子化的批量请求响应模式（事务）

redis 可以利用事务机制批量执行命令

#### 发布订阅模式

发布订阅模式是：一个客户端触发，多个客户端被动接收，通过服务器中转。

#### 脚本化批量执行（lua）

客户端向服务器端提交 lua 脚本，服务器执行该脚本。

##### 请求格式（两种）

###### 1.内联格式

```bash
[root@localhost bin]# telnet 127.0.0.1 6379
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
ping +PONG exists
name :1
```

###### 2.规范格式（协议响应格式）

```markdown
1、间隔符号，在 Linux 下是\r\n，在 Windows 下是\n
2、简单字符串 Simple Strings, 以 "+"加号 开头
3、错误 Errors, 以"-"减号 开头
4、整数型 Integer， 以 ":" 冒号开头
5、大字符串类型 Bulk Strings, 以 "$"美元符号开头，长度限制 512M
6、数组类型 Arrays，以 "\*"星号开头
用 SET 命令来举例说明 RESP 协议的格式。

##查看持久化
vim appendonly.aof
```

3 命令处理流程

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1651970633170-4cd60f4e-8de3-41df-a613-b9c51c8d78c0.png#averageHue=%23e5e6d3&clientId=u1ee8be83-248d-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub7275148&margin=%5Bobject%20Object%5D&name=image.png&originHeight=382&originWidth=763&originalType=url∶=1&rotation=0&showTitle=false&size=164922&status=done&style=none&taskId=ud956e76f-4096-45b1-8702-426b54a0786&title=)
**3.1 Server 启动监听 socket**
启动调用 initServer 方法：创建 eventLoop（事件机制），注册时间事件处理器，注册文件事件（socket）处理器，监听 socket 建立连接
**3.2 建立 client**
redis-cli 建立 socket,redis-server 为每个连接（socket）创建一个 Client 对象,创建文件事件监听 socket,指定事件处理函数
**3.3 读取 socket 数据到输入缓冲区**
从 client 中读取客户端的查询缓冲区内容。
**3.4 解析获取命令**
将输入缓冲区中的数据解析成对应的命令，判断是单条命令还是多条命令并调用相应的解析器解析
**3.5 执行命令**
解析成功后调用 processCommand 方法执行命令。调用 lookupCommand 方法获得对应的 redisCommand，检测当前 Redis 是否可以执行该命令，调用 call 方法真正执行命令

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1651970632474-2bbd411e-1000-4d25-ae4f-01345061679d.png#averageHue=%23f8f8f8&clientId=u1ee8be83-248d-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ua1dc0a4d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=699&originWidth=580&originalType=url∶=1&rotation=0&showTitle=false&size=90983&status=done&style=none&taskId=u87490dca-2f22-42d6-bc0a-16581911710&title=)

4 协议解析及处理
**4.1 协议解析：**
在 Redis 客户端键入命令后，Redis-cli 会把命令转化为 RESP 协议格式，然后发送给服务器。服务器再对协议进行解析。

- 解析命令请求参数数量
- 循环解析请求参数

**4.2 协议执行**
协议的执行包括命令的调用和返回结果，判断参数个数和取出的参数是否一致，RedisServer 解析完命令后,会调用函数 processCommand 处理该命令请求。quit 校验，如果是“quit”命令，直接返回并关闭客户端，命令语法校验，执行 lookupCommand，查找命令(set)，如果不存在则返回：“unknown command”错误。参数数目校验，参数数目和解析出来的参数个数要匹配，如果不匹配则返回：“wrong number of arguments”错误。此外还有权限校验，最大内存校验，集群校验，持久化校验等等。校验成功后，会调用 call 函数执行命令，并记录命令执行时间和调用次数，如果执行命令时间过长还要记录慢查询日志。执行命令后返回结果的类型不同则协议格式也不同，分为 5 类：状态回复、错误回复、整数回复、批量回复、多条批量回复
