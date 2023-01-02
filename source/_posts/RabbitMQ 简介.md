---
title: RabbitMQ 简介
urlname: toci4zptzbu15arg
date: '2022-12-26 11:02:36 +0800'
tags: []
categories: []
---

tags: []
categories: [RabbitMQ]
cover: ''

---

> RabbitMQ 是目前非常热门的一款消息中间件，不管是互联网行业还是传统行业都在大量地使用。
> RabbitMQ 凭借其
>
> - 高可靠
> - 易扩展
> - 高可用
> - 丰富的功能特性
>
> 受到越来越多企业的青睐。

## 什么是消息中间件

> 消息（Message）是指在应用间传送的数据。消息可以非常简单，比如只包含文本字符串、JSON 等，也可以很复杂，比如内嵌对象。
> 消息队列中间件（Message Queue Middleware，简称为 MQ）是指利用高效可靠的消息传递机制进行与平台无关的数据交流，并基于数据通信来进行分布式系统的集成。
> 通过提供消息传递和消息排队模型，它可以在分布式环境下扩展进程间的通信。
> 消息队列中间件，也可以称为消息队列或者消息中间件。
> 它一般有两种传递模式：
>
> - 点对点（P2P, Point-to-Point）模式
> - 发布/订阅（Pub/Sub）模式。
>
> 点对点模式是基于队列的，消息生产者发送消息到队列，消息消费者从队列中接收消息，队列的存在使得消息的异步传输成为可能。
> 发布订阅模式定义了如何向一个内容节点发布和订阅消息，这个内容节点称为主题（topic），主题可以认为是消息传递的中介，消息发布者将消息发布到某个主题，而消息订阅者则从主题中订阅消息。
> 主题使得消息的订阅者与消息的发布者互相保持独立，不需要进行接触即可保证消息的传递，发布/订阅模式在消息的一对多广播时采用。
> 目前开源的消息中间件有很多，比较主流的有 RabbitMQ、Kafka、ActiveMQ、RocketMQ 等。面向消息的中间件（简称为 MOM,Message Oriented Middleware）提供了以松散耦合的灵活方式集成应用程序的一种机制。
> 消息中间件适用于需要可靠的数据传送的分布式环境。
> 采用消息中间件的系统中，不同的对象之间通过传递消息来激活对方的事件，以完成相应的操作。
> 发送者将消息发送给消息服务器，消息服务器将消息存放在若干队列中，在合适的时候再将消息转发给接收者。消息中间件能在不同平台之间通信，它常被用来屏蔽各种平台及协议之间的特性，实现应用程序之间的协同，其优点在于能够在客户和服务器之间提供同步和异步的连接，并且在任何时刻都可以将消息进行传送或者存储转发，这也是它比远程过程调用更进步的原因。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672024166616-1260f062-ac82-4bed-8311-fe59459388f4.png#averageHue=%23f6f6f6&clientId=u0365c046-ec5a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=488&id=u8a215b56&margin=%5Bobject%20Object%5D&name=image.png&originHeight=975&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=187135&status=done&style=none&taskId=u18c4120a-2492-40c7-8d98-27f09888626&title=&width=720)

## 消息中间件的作用

> 消息中间件的作用可以概括如下
>
> - 解耦：
>   - 在项目启动之初来预测将来会碰到什么需求是极其困难的。消息中间件在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口，这允许你独立地扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束即可。
> - 冗余（存储）：
>   - 有些情况下，处理数据的过程会失败。消息中间件可以把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。在把一个消息从消息中间件中删除之前，需要你的处理系统明确地指出该消息已经被处理完成，从而确保你的数据被安全地保存直到你使用完毕。
> - 扩展性
>   - 因为消息中间件解耦了应用的处理过程，所以提高消息入队和处理的效率是很容易的，只要另外增加处理过程即可，不需要改变代码，也不需要调节参数
> - 削峰
>   - 在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果以能处理这类峰值为标准而投入资源，无疑是巨大的浪费。使用消息中间件能够使关键组件支撑突发访问压力，不会因为突发的超负荷请求而完全崩溃。
> - 可恢复性
>   - 当系统一部分组件失效时，不会影响到整个系统。消息中间件降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入消息中间件中的消息仍然可以在系统恢复后进行处理
> - 顺序保证
>   - 在大多数使用场景下，数据处理的顺序很重要，大部分消息中间件支持一定程度上的顺序性。
> - 缓冲
>   - 任何重要的系统中，都会存在需要不同处理时间的元素。消息中间件通过一个缓冲层来帮助任务最高效率地执行，写入消息中间件的处理会尽可能快速。该缓冲层有助于控制和优化数据流经过系统的速度。
> - 异步通信
>   - 在很多时候应用不想也不需要立即处理消息。消息中间件提供了异步处理机制，允许应用把一些消息放入消息中间件中，但并不立即处理它，在之后需要的时候再慢慢处理

## 起源

> RabbitMQ 是采用 Erlang 语言实现 AMQP（Advanced Message Queuing Protocol，高级消息队列协议）的消息中间件，它最初起源于金融系统，用于在分布式系统中存储转发消息。
> RabbitMQ 最初版本实现了 AMQP 的一个关键特性：使用协议本身就可以对队列和交换器（Exchange）这样的资源进行配置。对于商业 MQ 供应商来说，资源配置需要通过管理终端的特定工具才能完成。RabbitMQ 的资源配置能力使其成为构建分布式应用的最完美的通信总线，特别有助于充分利用基于云的资源和进行快速开发。

具体特点可以概括为以下几点。
✧ 可靠性：RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认及发布确认等。
✧ 灵活的路由：在消息进入队列之前，通过交换器来路由消息。对于典型的路由功能， RabbitMQ 已经提供了一些内置的交换器来实现。针对更复杂的路由功能，可以将多个交换器绑定在一起，也可以通过插件机制来实现自己的交换器。
✧ 扩展性：多个 RabbitMQ 节点可以组成一个集群，也可以根据实际业务情况动态地扩展集群中节点。
✧ 高可用性：队列可以在集群中的机器上设置镜像，使得在部分节点出现问题的情况下队列仍然可用。
✧ 多种协议：RabbitMQ 除了原生支持 AMQP 协议，还支持 STOMP、MQTT 等多种消息中间件协议。
✧ 多语言客户端：RabbitMQ 几乎支持所有常用语言，比如 Java、Python、Ruby、PHP、C#、JavaScript 等。
✧ 管理界面：RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息、集群中的节点等。
✧ 插件机制：RabbitMQ 提供了许多插件，以实现从多方面进行扩展，当然也可以编写自己的插件

## 安装

### 安装 Erlang

```bash
tar zxvf otp_src_19.3.tar.gz
cd otp_src_19.3
./configure --prefix=/opt/erlang

#如果出现类似关键报错信息：No curses library functions found。那么此时需要安装ncurses
yum install ncurses-devel
make && make install

#第四步，修改/etc/profile配置文件，添加下面的环境变量：
ERLANG_HOME=/opt/erlang
export PATH=$PATH:$ERLA
NG_HOME/binexport ERLANG_HOME

source /etc/profile

```

### 安装 RabbitMQ

> 官网下载地址：[http://www.rabbitmq.com/releases/rabbitmq-server/](http://www.rabbitmq.com/releases/rabbitmq-server/)。本书撰稿时的最新版本为 3.6.12，本书示例大多采用同一系列的 3.6.x 版本

```bash
#这里选择将RabbitMQ安装到与Erlang同一个目录（/opt）下面：
tar zvxf rabbitmq-server-generic-unix-3.6.10.tar.gz -C /opt
cd /opt
mv rabbitmq_server-3.6.10 rabbitmq

#同样修改/etc/profile文件，添加下面的环境变量：
export PATH=$PATH:/opt/rabbitmq/sbin
export RABBITMQ_HOME=/opt/rabbitmq
```

## 运行

> 在修改了/etc/profile 配置文件之后，可以任意打开一个 Shell 窗口，输入如下命令以运行 RabbitMQ 服务：

```bash
rabbitmq-server -detached
#在 rabbitmq-server 命令后面添加一个“-detached”参数是为了能够让 RabbitMQ服务以守护进程的方式在后台运行，这样就不会因为当前Shell窗口的关闭而影响服务。

#运行rabbitmqctl status命令查看RabbitMQ是否正常启动
rabbitmqctl status

#当然也可以通过 rabbitmqctl cluster_status命令来查看集群信息，
rabbitmqctl cluster_status

```

## 使用

> 默认情况下，访问 RabbitMQ 服务的用户名和密码都是“guest”，这个账户有限制，默认只能通过本地网络（如 localhost）访问，远程网络访问受限，所以在实现生产和消费消息之前，需要另外添加一个用户，并设置相应的访问权限。

### 创建用户

```bash
#添加新用户，用户名为“root”，密码为“root123”
rabbitmqctl add_user root root
#为root用户设置所有权限：
rabbitmqctl set_permissions -p / root ".＊" ".＊" ".＊"
#设置root用户为管理员角色
rabbitmqctl set_user_tags root administrator

```

### hello world

```java
package com.zzh.rabbitmq.demo;
import com.rabbitmq.client.Chan
nel;import com.rabbitmq.client.Conn
ection;import com.rabbitmq.client.Conn
ectionFactory;import com.rabbitmq.client.Mess
ageProperties;import java.io.IOException;import java.util.concurrent.Tim
eoutException;public class RabbitProducer {    private static final String
 EXCHANGE_NAME = "exchange_demo";    private static final String
 ROUTING_KEY = "routingkey_demo";    private static final String
 QUEUE_NAME = "queue_demo";    private static final String
 IP_ADDRESS = "192.168.0.2";    private static final int PORT = 5672; //RabbitMQ服务端默认端口号为5672
public static void main(String[] args) throws IOException,          TimeoutException, Int
erruptedException {        ConnectionFactory fact
o
ry = new ConnectionFactory();        factory.setHost(IP_ADD
R
ESS);        factory.setPort(PORT);
        factory.setUsername("r
o
ot");        factory.setPassword("r
o
ot123");        Connection connection
=
 factory.newConnection(); //创建连接        Channel channel = conn
e
ction.createChannel(); //创建信道        //创建一个type="direct"、持久
化
的、非自动删除的交换器        channel.exchangeDeclar
e
(EXCHANGE_NAME, "direct", true, fals
e, null);        //创建一个持久化、非排他的、非自动删除的队
列
        channel.queueDeclare(Q
U
EUE_NAME, true, false, false, null);
        //将交换器与队列通过路由键绑定        channel.queueBind(QUEU
E
_NAME, EXCHANGE_NAME, ROUTING_KEY);        //发送一条持久化的消息：hello wor
l
d!        String message = "Hell
o
 World! ";        channel.basicPublish(E
X
CHANGE_NAME, ROUTING_KEY,              MessagePropertie
s
.PERSISTENT_TEXT_PLAIN,              message.getBytes
(
));        //关闭资源        channel.close();        connection.close();    }}
```
