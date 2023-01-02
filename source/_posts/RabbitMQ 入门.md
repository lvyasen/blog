---
title: RabbitMQ 入门
urlname: sekdvlwa6017d68h
date: '2022-12-26 11:41:19 +0800'
tags: []
categories: []
---

## 相关概念介绍

> RabbitMQ 整体上是一个生产者与消费者模型，主要负责接收、存储和转发消息。
> 可以把消息传递的过程想象成：当你将一个包裹送到邮局，邮局会暂存并最终将邮件通过邮递员送到收件人的手上，RabbitMQ 就好比由邮局、邮箱和邮递员组成的一个系统。
> 从计算机术语层面来说， RabbitMQ 模型更像是一种交换机模型

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672026194767-82f854d2-6131-475c-81c0-13ab43f36dc8.png#averageHue=%23f5f5f5&clientId=u56fd7eec-98f2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=365&id=u64831ca3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=730&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=277060&status=done&style=none&taskId=uc8ea785a-11f7-41a1-8f90-892e9fdcb98&title=&width=720)

### 生产者和消费者

> Producer：生产者，就是投递消息的一方。
>
> - 生产者创建消息，然后发布到 RabbitMQ 中。
> - 消息一般可以包含 2 个部分：消息体和标签（Label）。消息体也可以称之为 payload，在实际应用中，消息体一般是一个带有业务逻辑结构的数据，比如一个 JSON 字符串。当然可以进一步对这个消息体进行序列化操作。
> - 消息的标签用来表述这条消息，比如一个交换器的名称和一个路由键。生产者把消息交由 RabbitMQ, RabbitMQ 之后会根据标签把消息发送给感兴趣的消费者（Consumer）。
>
> Consumer：消费者，就是接收消息的一方。
>
> - 消费者连接到 RabbitMQ 服务器，并订阅到队列上。当消费者消费一条消息时，只是消费消息的消息体（payload）。
> - 在消息路由的过程中，消息的标签会丢弃，存入到队列中的消息只有消息体，消费者也只会消费到消息体，也就不知道消息的生产者是谁，当然消费者也不需要知道。
>
> Broker：消息中间件的服务节点。
>
> - 对于 RabbitMQ 来说，一个 RabbitMQ Broker 可以简单地看作一个 RabbitMQ 服务节点，或者 RabbitMQ 服务实例。
> - 大多数情况下也可以将一个 RabbitMQ Broker 看作一台 RabbitMQ 服

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672026327622-cfe33c09-d68b-401c-b71e-367199c48376.png#averageHue=%23f4f4f4&clientId=u56fd7eec-98f2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=402&id=ub8a0073d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=803&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=294287&status=done&style=none&taskId=u3823bd31-0c29-4df8-a051-8949da04ef0&title=&width=720)

### 队列

> Queue：队列，是 RabbitMQ 的内部对象，用于存储消息。
> RabbitMQ 中消息都只能存储在队列中，这一点和 Kafka 这种消息中间件相反。
> Kafka 将消息存储在 topic（主题）这个逻辑层面，而相对应的队列逻辑只是 topic 实际存储文件中的位移标识。
> RabbitMQ 的生产者生产消息并最终投递到队列中，消费者可以从队列中获取消息并消费。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672026377236-623a0e49-aed7-4cf6-94f4-a6ccff03a8f2.png#averageHue=%23f7f7f7&clientId=u56fd7eec-98f2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=193&id=u235f6fc2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=386&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=27307&status=done&style=none&taskId=u0cb3080f-1f0e-4039-911f-94681a4bc96&title=&width=720)
多个消费者可以订阅同一个队列，这时队列中的消息会被平均分摊（Round-Robin，即轮询）给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672026429707-e26998cf-54c8-4ff6-9575-570e9632fe0e.png#averageHue=%23f8f8f8&clientId=u56fd7eec-98f2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=233&id=uac6ff2a6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=465&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=99452&status=done&style=none&taskId=u58baa699-f555-4fac-a6eb-9775d8fb843&title=&width=720)

### 交换器、路由键、绑定

> Exchange：交换器。
> 我们暂时可以理解成生产者将消息投递到队列中，实际上这个在 RabbitMQ 中不会发生。
> 真实情况是，生产者将消息发送到 Exchange（交换器，通常也可以用大写的“X”来表示），由交换器将消息路由到一个或者多个队列中。
> 如果路由不到，或许会返回给生产者，或许直接丢弃。
> 这里可以将 RabbitMQ 中的交换器看作一个简单的实体。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672026520165-3bcb47c7-87c4-449a-b0f9-96e42f6d7527.png#averageHue=%23f7f7f7&clientId=u56fd7eec-98f2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=317&id=u2e8fb131&margin=%5Bobject%20Object%5D&name=image.png&originHeight=634&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=108451&status=done&style=none&taskId=ub221afc2-9290-4d5c-91ae-f1f2a8e4ee9&title=&width=720)
RabbitMQ 中的交换器有四种类型，不同的类型有着不同的路由策略。
RoutingKey：路由键。生产者将消息发给交换器的时候，一般会指定一个 RoutingKey，用来指定这个消息的路由规则，而这个 Routing Key 需要与交换器类型和绑定键（BindingKey）联合使用才能最终生效。
在交换器类型和绑定键（BindingKey）固定的情况下，生产者可以在发送消息给交换器时，通过指定 RoutingKey 来决定消息流向哪里。
Binding：绑定。RabbitMQ 中通过绑定将交换器与队列关联起来，在绑定的时候一般会指定一个绑定键（BindingKey），这样 RabbitMQ 就知道如何正确地将消息路由到队列了，

## AMQP 协议介绍
