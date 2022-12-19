---
title: Go 的诞生
urlname: yh1z71rxhdb9so9n
date: '2022-12-19 14:36:24 +0800'
tags:
  - go
categories:
  - Go
---

> 2007 年 9 月 20 日 在谷歌，由 Rob Pike、Ken Thompson、Robert Griesemer（ 罗伯特·格瑞史莫），为了解决 C++，编译速度慢、不便支持并发。
> 2007 年 9 月 25 日，Rob 命名为 Go。
> 2008 年年初 unix 之父 Ken Thompson 实现了第一版 go 编译器。编译器先将 go 代码转为 c 代码，再由 c 转为二进制文件。
> go 的语法参考了 c 语言，Go 是“C 家族语言”的一个分支；些并发的思想则来自受到 Tony Hoare 教授 CSP 理论影响的编程语言

**优点：**

- **编译速度块**
- **高性能**
- **原生支持并发**
- **强大的社区**
- **内置垃圾回收**
- **简单**

## Go 语言原生支持并发的设计哲学体现在以下几点

> Go 果断放弃了传统的基于操作系统线程的并发模型，而采用了用户层轻量级线程或者说是类协程（coroutine），Go 将之称为 goroutine。
> goroutine 占用的资源非常少，Go 运行时默认为每个 goroutine 分配的栈空间仅 2KB。
> goroutine 调度的切换也不用陷入（trap）操作系统内核层完成，代价很低

- Go 语言采用轻量级协程并发模型，使得 Go 应用在面向多核硬件时更具可扩展性。
- Go 语言为开发者提供的支持并发的语法元素和机。

## 并发并行

- 并发是有关结构的，它是一种将一个程序分解成多个小片段并且每个小片段都可以独立执行的程序设计方法；并发程序的小片段之间一般存在通信联系并且通过通信相互协作。
- 并行是有关执行的，它表示同时进行一些计算任务

## 如何解决工程领域规模化所带来的问题的

### 语言

> Go 语言是一门简单的语言，简单意味着可读性好，容易理解，容易上手，容易修复错误，节省开发者时间，提升开发者间的沟通效率。

- 重新设计编译单元和目标文件格式，实现 Go 源码快速构建，将大工程的构建时间缩短到接近于动态语言的交互式解释的编译时间。
- 如果源文件导入了它不使用的包，则程序将无法编译。
- 去除包的循环依赖。
- 包路径是唯一的，而包名不必是唯一的。
- 故意不支持默认函数参数。
- 字母大小写定义标识符可见性。
- 去除指针算术，去除隐式类型转换等。
- 内置垃圾收集。
- 内置并发支持。
- 增加类型别名。

### 标准库

> Go 被称为“自带电池”（battery-included）的编程语言。“自带电池”原指购买了电子设备后，在包装盒中包含了电池，电子设备可以开箱即用，无须再单独购买电池。
> net/http、crypto/xx、encoding/xx

### 工具链

> 开发人员在做工程的过程中需要使用工具。

- 构建和运行：go build/go run
- 依赖包查看与获取：go list/go get/go mod xx
- 编辑辅助格式化：go fmt/gofmt
- 文档查看：go doc/godoc
- 单元测试/基准测试/测试覆盖率：go test
- 代码静态分析：go vet
- 性能剖析与跟踪结果查看：go tool pprof/go tool trace
- 升级到新 Go 版本 API 的辅助工具：go tool fix
- 报告 Go 语言 bug：go bug

## 项目结构

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1669603349212-da36d1ab-3a60-4a51-9324-83e9cc7ec24b.png#averageHue=%23e0dfdd&clientId=ube363a93-e40f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=257&id=ud62c21fb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1212&originWidth=1416&originalType=binary∶=1&rotation=0&showTitle=false&size=119904&status=done&style=none&taskId=u4013091d-ee0d-4e45-a393-662d151e706&title=&width=300)
