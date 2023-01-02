---
title: Go & PHP
urlname: gkw82og55nmleogg
date: '2022-12-30 15:26:32 +0800'
tags: []
categories: []
---

## 相关文献参考

> [CSP 通讯简述](http://c.biancheng.net/view/5111.html)

## go 语言对比 PHP 优缺点

- Go 是静态语言，PHP 是动态脚本语言。
- Go 需要编译，PHP 无需编译。
- Go 打包部署后，别人无法看到源码；php 能看到源码，如果黑进主机 php 源码有泄漏风险。
- Go 语言天然支持并发，内置 goroutine 启动一个协程只需要 2KB，而一个线程则需要 2MB。
- Go 语言采用 CSP 并发模型，虽然并未完全实现 CSP 并发模型理论，仅仅实现了 process 和 channel，process 就是 goroutine，每个 goroutine 通过 channel 通讯实现数据共享。
