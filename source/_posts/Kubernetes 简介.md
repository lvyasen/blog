---
title: Kubernetes 简介
urlname: btfdg0gr6gvb921a
date: '2022-12-31 13:51:00 +0800'
tags: []
categories: []
---

tags: []
categories: [Kubernetes]
cover: ''

---

> [中文官网](https://kubernetes.io/zh-cn/) > [快速上手教程](https://k8s.easydoc.net/docs/dRiQjyTY/28366845/6GiNOzyZ/9EX8Cp45) & [配套课程](https://www.bilibili.com/video/BV1Tg411P7EB?p=2&vd_source=562b0a9746c823abc601464476e85cdf)

## 什么是 Kubernetes？

> 它是一个为 **容器化** 应用提供集群部署和管理的开源工具，由 Google 开发。
> **Kubernetes** 这个名字源于希腊语，意为“舵手”或“飞行员”。k8s 这个缩写是因为 k 和 s 之间有八个字符的关系。 Google 在 2014 年开源了 Kubernetes 项目。

### 主要特性

- 高可用，不宕机，自动灾难恢复
- 灰度更新，不影响业务正常运转
- 一键回滚到历史版本
- 方便的伸缩扩展（应用伸缩，机器加减）、提供负载均衡
- 有一个完善的生态

## 不同应用的部署方式

### 传统方式部署

> 应用直接在物理机上部署，机器资源分配不好控制，出现 Bug 时，可能机器的大部分资源被某个应用占用，导致其他应用无法正常运行，无法做到应用隔离。

### 虚拟机部署

> 在单个物理机上运行多个虚拟机，每个虚拟机都是完整独立的系统，性能损耗大。

### 容器部署

> 所有容器共享主机的系统，轻量级的虚拟机，性能损耗小，资源隔离，CPU 和内存可按需分配。

## 什么时候需要 Kubernetes？

> 当你的应用只是跑在一台机器，直接一个 docker + docker-compose 就够了，方便轻松；
> 当你的应用需要跑在 3，4 台机器上，你依旧可以每台机器单独配置运行环境 + 负载均衡器；
> 当你应用访问数不断增加，机器逐渐增加到十几台、上百台、上千台时，每次加机器、软件更新、版本回滚，都会变得非常麻烦、痛不欲生，再也不能好好的摸鱼了，人生浪费在那些没技术含量的重复性工作上。
> 这时候，Kubernetes 就可以一展身手了，让你轻松管理百万千万台机器的集群。“谈笑间，樯橹灰飞烟灭”，享受着一手掌控所有，年薪百万指日可待。
> Kubernetes 可以为你提供集中式的管理集群机器和应用，加机器、版本升级、版本回滚，那都是一个命令就搞定的事，不停机的灰度更新，确保高可用、高性能、高扩展。

## Kubernetes 集群架构

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672466085015-45f135db-43c3-4dcd-9a54-dc320a6bad5d.png#averageHue=%23f7f7f7&clientId=u1a264b23-3005-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=229&id=u5ce6bf04&margin=%5Bobject%20Object%5D&name=image.png&originHeight=457&originWidth=514&originalType=binary∶=1&rotation=0&showTitle=false&size=34801&status=done&style=none&taskId=uef87c07f-c155-4e70-8cb9-3c8044d2443&title=&width=257)

### master

> 主节点，控制平台，不需要很高性能，不跑任务，通常一个就行了，也可以开多个主节点来提高集群可用度。

### worker

> 工作节点，可以是虚拟机或物理计算机，任务都在这里跑，机器性能需要好点；通常都有很多个，可以不断加机器扩大集群；每个工作节点由主节点管理。

### pod

> 豆荚，K8S 调度、管理的最小单位，一个 Pod 可以包含一个或多个容器，每个 Pod 有自己的虚拟 IP。一个工作节点可以有多个 pod，主节点会考量负载自动调度 pod 到哪个节点运行。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672466155818-a9030ff6-2b82-45ea-af25-a0ee164a0611.png#averageHue=%23f5f5f5&clientId=u1a264b23-3005-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=305&id=ub7fc771a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=610&originWidth=856&originalType=binary∶=1&rotation=0&showTitle=false&size=511147&status=done&style=none&taskId=u149c7900-a410-4151-9d9d-b15682fa85d&title=&width=428)

### Kubernetes 组件

> kube-apiserver API 服务器，公开了 Kubernetes API
> etcd 键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库
> kube-scheduler 调度 Pod 到哪个节点运行
> kube-controller 集群控制器
> cloud-controller 与云服务商交互

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672466220618-fc580654-0401-4b31-9975-fb1c9b90d625.png#averageHue=%23dcdcdc&clientId=u1a264b23-3005-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=226&id=ue1ccf45b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=452&originWidth=944&originalType=binary∶=1&rotation=0&showTitle=false&size=71375&status=done&style=none&taskId=u194f3a3f-4cf7-4c82-8068-af565804cd4&title=&width=472)
