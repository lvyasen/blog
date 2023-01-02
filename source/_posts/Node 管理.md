---
title: Node 管理
urlname: fc4geeo0g93czc33
date: '2022-12-23 09:24:32 +0800'
tags: []
categories: []
---

## 版本管理

### 安装 N 管理工具

```bash
# 查看当前版本
node -v
# 清理本地包缓存
npm cache clean -f
# 安装
npm i -g n
# 查看n是否安装成功
n -V
```

### 更新 node 版本

```bash
n stable // 把当前系统的 Node 更新成最新的 “稳定版本”
n lts // 长期支持版
n latest // 最新版
n 16.13.1 // 指定安装版本
```

### 查看仓库版本

```bash
n ls-remote
```

### 安装指定版本

```bash
n ls-remote  14.18.0

//升级到最新稳定版，可能需要 root 权限
n stable
```

## 镜像

### 查看镜像

```bash
npm get registry
```

### 设置阿里镜像

```bash
//清除镜像
npm config set proxy null &&
npm config set https-proxy null

//设置镜像
npm config set registry http://registry.npm.taobao.org/
//下面是其他资源镜像
npm config set registry http://r.cnpmjs.org/

npm config set registry http://registry.cnpmjs.org/
```

### 关闭 https

```bash
npm config set strict-ssl false
```
