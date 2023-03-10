---
title: Redis 限流
urlname: hikgrgmruimtzdde
date: '2022-12-15 08:40:51 +0800'
tags:
  - redis
categories:
  - Redis
---

## 基于字符串 setnx

> 如 30s 内限定 10 个请求，过期时间设定在 30s，当过来一个请求判断设置的 key 的 val 是否超过 10 个请求

```shell
if(get(request)<10){
	incr request 1;
}else{
	set request 10 ex 30 nx;
}

```

## 基于有序集合 zset 滑动窗口限流

> - 用户的请求用一个 zset 存储，score、value 存储时间戳
> - 只保留滑动窗口内的数据，其他的删除

```shell
zremrangeByScore requests 0 1671066862 #先移除当前时间戳-指定时间秒数
zcard requests #统计元素个数是否溢出，没有溢出请求进来
```

## 基于 List 实现令牌桶

> 服务端使用定时脚本生产令牌到 list 中
> 客户端每次从右侧取出令牌
