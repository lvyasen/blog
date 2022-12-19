---
title: MySQL CPU 飙高
urlname: kyqop200gekng2ua
date: '2022-12-06 08:23:57 +0800'
tags:
  - mysql
categories:
  - mysql
---

> 正常使用过程中，如出现 CPU 飙高严重，通常是低效 SQL 语句导致的，处理过程如

- show processlist 语句，查找负荷最重的 sql 语句。
- 优化存在问题的 sql 语句，如建立索引。
- 打开慢查询日志，将执行时间过长且占用资源过多的语句拿来 explain 分析。CPU 过高，多数是因为 order by 和 group by 语句所致，找到问题并慢慢优化改进。
- 考虑定期优化文件及索引。
- 定期分析表，使用 optimize table （尽管一张表删除了许多数据，但是这张表表的数据文件和索引文件却奇怪的没有变小。这是因为 mysql 在删除数据（特别是有 Text 和 BLOB）的时候，会留下许多的数据空洞，这些空洞会占据原来数据的空间，所以文件的大小没有改变。这些空洞在以后插入数据的时候可能会被再度利用起来，当然也有可能一直存在。这种空洞不仅额外增加了存储代价，同时也因为数据碎片化降低了表的扫描效率，在 OPTIMIZE TABLE 运行过程中，MySQL 会锁定表）。show table news，对于 myisam 可以直接使用 optimize table table.name, 当是 InnoDB 引擎时，会报“Table does not support optimize, doing recreate + analyze instead”，一般情况下，由 myisam 转成 innodb，会用 alter table table.name engine='innodb'进行转换，优化也可以用这个。所以当是 InnoDB 引擎时我们就用 alter table table.name engine='innodb'来代替 optimize 做优化就可以。查看前后效果可以使用 show table status 命令,例如 show table status from [database] like '[table_name]';返回结果中的 data_free 即为空洞所占据的存储空间
- 检查是否存在锁问题。
