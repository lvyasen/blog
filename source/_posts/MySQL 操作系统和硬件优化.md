---
title: MySQL 操作系统和硬件优化
urlname: mva4es48iaxw5rd7
date: '2022-11-29 09:36:18 +0800'
tags:
  - mysql
categories:
  - mysql
---

> 常见的瓶颈是 CPU 耗尽，当 mysql 执行并行执行太多查询，或者少量查询在 CPU 上运行太长时间时，就会发生 CPU 饱和。

## 如何为 mysql 选择 CPU

> 可以通过 cpu 使用率来确定工作负载是否受 cpu 限制，但是不要看 cpu 整体负载，
> 而是查看最重要查询的 cpu 使用率和 IO 之间的平衡。并注意 cpu 负载是否均匀
> 服务器要达到两个目标
>
> - 低延迟：
> - 高吞吐量
