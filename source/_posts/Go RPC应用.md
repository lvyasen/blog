---
title: Go RPC应用
urlname: vfru1m6thqtskf0i
date: '2022-12-20 16:10:12 +0800'
tags: []
categories: []
---

tags: []
categories: []
cover: ''

---

> 在 gRPC 里客户端应用可以像调用本地方法一样直接调用另一台机器上服务端应用的方法，这样我们就很容易创建分布式应用和服务。
> 跟其他 RPC 系统类似，gRPC 也是基于以下理念：首先定义一个服务，定义能够被远程调用的方法（包含参数和返回类型）。
> 在服务端实现这个方法，并运行一个 gRPC 服务器来处理客户端调用。
> 在客户端拥有一个存根，这个存根就是长得像服务端一样的方法（但是没有具体实现），客户端通过这个存根调用服务端的方法。
> gRPC 基于 HTTP/2 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特性。这些特性使得其在移动设备上表现更好，更省电和节省空间占用。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672452779668-42b12f36-7923-437e-9103-e9dcf2fb4634.png#averageHue=%23fefefc&clientId=ud204eaa6-653d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=453&id=ucc9487c2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=905&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=356232&status=done&style=none&taskId=ub2243e7f-fe83-4121-b22a-9378975f1fa&title=&width=720)

### 服务方法

- **Unary RPCs，一元 RPC**。客户端发送一个请求到服务端，服务端响应一个请求。
  - rpc getUser (User) returns (User) {}
- **Server streaming RPCs，服务端流 RPC**。客户端发送一个请求到服务端，获取到一个流去连续读取返回的消息，直到消息全部获取。gRPC 保证单个请求的消息顺序。
  - rpc getUsers (User) returns (stream User) {}
- **Client streaming RPCs，客户端流 RPC**。客户端给服务器通过流写入连续的消息，一旦客户端完成了消息写入，就等待服务端读取完成然后返回一个响应。同时 gRPC 也会保证单个请求的消息顺序。
  - rpc saveUsers (stream User) returns (User) {}
- **Bidirectional streaming RPCs，双向流**。客户端和服务端都可以通过 read-write 流发送一个连续的消息。两个流之间的操作是相互独立的。所以，客户端和服务端可以同时进行流的读写。
  - rpc saveUsers (stream User) returns (stream User) {}
