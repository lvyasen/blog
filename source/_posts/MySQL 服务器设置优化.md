---
title: MySQL 服务器设置优化
urlname: kpsav2r5l5m63ucx
date: '2022-11-24 15:35:29 +0800'
tags:
  - mysql
categories:
  - mysql
---

> 首先确保 innodb 缓冲池和日志大小等基本设置是合适的。

## Mysql 配置是如何工作的？

> unix 系统上，配置通常在 /etc/my.cnf 或 /etc/mysql/my.cnf
> 如果不知道在哪里，which mysqld，/usr/sbin/mysqld --verbose --help | grep -A 1 'Default options'
> 语法：单词全部小写，单词之间使用 - 或 \_，分割

## 全局内存

- innodb*buffer_pool_size：用来缓存数据和索引数据。默认是 134217728 字节，约为 128MB。5.75 版本后不重启数据库可以生效。一般设为可用内存的 50%~80%。使用 show global status like 'innodb_buffer_pool*%'; 其中 innodb_buffer_pool_wait_free 较大时调整 innodb_buffer_pool_size。
- innodb_additional_mem_pool_size：存储数据字典和其他内部结构的内存池大小。默认 8 MB，通常不用太大够用就行，与表结构复杂程度有关。如果不够用错误日志会出现一条告警。
- innodb_log_buffer_size：日志缓冲，提高 redo 写入效率。
- key_buffer_size：索引缓存大小。
- query_cache_size：查询高速缓冲。
- table_definition_cache：表定义描述缓存。
