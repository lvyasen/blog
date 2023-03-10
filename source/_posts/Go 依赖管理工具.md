---
title: Go 依赖管理工具
urlname: nwhcou8vf3cbs6ci
date: '2022-12-23 22:10:23 +0800'
tags: []
categories: []
---

tags: [依赖管理  ]
categories: [Go]
cover: ''

---

## 包安装

### 下载单个包

> 无法有效管理依赖包的不同版本，go get 只能获取到最新版本的包，由于无法指定依赖版本，依赖包的源码仓库有变动，必然会造成意外影响(更新依赖发现不能编译了，多人开发依赖版本不一致导致的 bug 等)
> 无法达成同一个项目，依赖包的不同版本，比如，你要做 ElasticSearch 迁移，从 ElasticSearch6 迁移到 ElasticSearch7，这两个版本的 client sdk 有不兼容。想象下，如果不支持同时依赖多个版本，对于开发者来说，要么在依赖包的代码仓库层做文章(比如，不同仓库，不同目录等)，要么下载依赖包后，在包的存储目录上处理，效率太低

```bash
go get <包地址>
```

### 下载相关依赖包

```bash
go install
```

### 编译包

```bash
go build
```
