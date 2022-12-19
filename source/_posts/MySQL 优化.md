---
title: MySQL 优化
urlname: colw2qbga54fnuvg
date: '2022-11-19 09:06:28 +0800'
tags:
  - mysql
categories:
  - mysql
---

> 所有列有默认值，当在严格模式时必须为 not null 列指定默认值。

可以使用 mysql 基准套件

## 如何获取慢查询？

- 使用 mysql 的 slow log 或 show processlist 命令获取。
- 通过业务记录访问日志。
- 通过 mysql general log。
- 云服务厂商提供的审计日志。

### show processlist （显示正在运行的线程）

> 该命令显示的信息来自 information_schema.processlist

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1668823807461-3e714abd-7616-46c7-acf5-a81cff7d96e7.png#averageHue=%230d2b35&clientId=u193c207c-61fe-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=210&id=u4b2a4230&margin=%5Bobject%20Object%5D&name=image.png&originHeight=420&originWidth=2318&originalType=binary∶=1&rotation=0&showTitle=false&size=114238&status=done&style=none&taskId=udb5856da-87b5-465f-a83f-3fef8fde4a0&title=&width=1159)

- id 线程唯一标识，如果发现线程有问题，可以通过 kill id 将该线程杀掉，
- user 指启动这个线程的用户。
- host 记录了请求客户端的 ip 和端口号。
- db 当前执行的命令在那个数据库上。
- command 指此时此刻线程上执行的命令。
- time 该线程处于当前状态的时间；
- stat 线程的状态。
- info 一般记录线程执行的语句，默认只显示 100 个字符，如果看全部信息使用 show full processlist。

#### command 的值

- binlog dump：主节点正将二进制日志同步到从节点。
- change user：正在执行一个 change-user 操作。
- close stmt：正在关闭一个 Prepared Statement 对象
- connect：一个从节点连接到了主节点。
- connect out：一个从节点正在连接主节点。
- create db：正在执行一个创建数据库的操作。
- Create DB: 正在执行一个 create-database 的操作
- Daemon: 服务器内部线程，而不是来自客户端的链接
- Debug: 线程正在生成调试信息
- Delayed Insert: 该线程是一个延迟插入的处理程序
- Drop DB: 正在执行一个 drop-database 的操作
- Execute: 正在执行一个 Prepared Statement
- Fetch: 正在从 Prepared Statement 中获取执行结果
- Field List: 正在获取表的列信息
- Init DB: 该线程正在选取一个默认的数据库
- Kill : 正在执行 kill 语句，杀死指定线程
- Long Data: 正在从 Prepared Statement 中检索 long data
- Ping: 正在处理 server-ping 的请求
- Prepare: 该线程正在准备一个 Prepared Statement
- ProcessList: 该线程正在生成服务器线程相关信息
- Query: 该线程正在执行一个语句
- Quit: 该线程正在退出

## 如何知道在哪发生了性能问题？

> - 使用 explain select 语句分析慢日志+show profiles；
> - performance schema
> - trace

### Explain （获取 sql 执行计划）

可以获取以下信息：

- 多张表 join 的读取顺序。
- 每张表的预估扫描行数。
- SQL 的查询类型。
- 可能用到的索引情况。
- 实际用到的索引。
- 额外信息（是否使用排序、是否使用覆盖索引、）

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1668844583078-1ee5348a-1c17-4eda-b5b3-ebf9e93d2b87.png#averageHue=%230d2a35&clientId=u193c207c-61fe-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=361&id=u498bd401&margin=%5Bobject%20Object%5D&name=image.png&originHeight=722&originWidth=1372&originalType=binary∶=1&rotation=0&showTitle=false&size=89575&status=done&style=none&taskId=ua9829361-001c-4735-8bb0-1922039e1aa&title=&width=686)

#### id ：单个语句标识符。这是 select 语句的查询序号，如果是 union 语句可能为 null（如果 id 相同，执行顺从上到下；如果是子查询，id 序号会递增，id 值越大越会被优先执行）。

#### select_type 查询类型（总体分为简单查询和复杂查询）。

- SIMPLE：**简单 select，**不使用 union 语句或子查询。
- **PRIMARY**：**如果包含关联查询或子查询**，那么最外层查询部分会被标记为 PRIMARY。
- **union：union** 关联查询 all 语句中第二条语句或后面的 select 子句。
- **dependent subquery：**在 union 中，子查询的第一条 select 语句，依赖外部查询。
- **dependent** **union**：与 UNION 相同，但是依赖外部查询。
- subquery：子查询中的第一条 select 子句，结果不依赖外部查询。
- derived：派生表的 select 子句，from 的子查询。
- uncacheable subquery：无法缓存其结果的子查询，必须对外部每一行进行重新求值。

```sql
explain select * from test1 where videoid in ( #该查询在最外侧，且包含关联查询，所以他的 select type 为 primary
  select videoid from test2 where taskid=9812 #该查询是子查询的第一条语句，所以 select_type 为 dependent subquery
  union select videoid from test1 where memid=234 #该select_type 为 dependent union
                                             ) ;#该查询为整体 union 结果所以，select_type 为 union
#
#该查询分为 4 步；

```

#### type 联接（join）类型。

> 性能从差到好

1. ALL：全表扫描，myql 将遍历全表。
2. index：全索引扫描。
3. range：只检索给定范围的行，使用一个索引选择行。
4. unique_subquery：in 形式的子查询，子查询中返回不重复的唯一值（如子查询结果为 id）。
5. index_subquery：类似于 unique_subquery ，in 形式的子查询，单子查询返回非唯一值。
6. index_merge：该连接使用了索引合并优化方法。
7. ref：join 时非主键和非唯一索引的等值扫描。
8. ref_or_null：类似于 ref ，但是在等值扫描时需要扫描包含 null 值得行。
9. fulltext：join 时使用全文索引。
10. eq_ref：类似于 ref ，但使用的索引是唯一索引，对于每个索引键值，表中只有一条匹配记录。
11. const、system：常量级访问，如将主键置于 where 条件中，mysql 就能将该查询转换为一个常量，system 是 const 类型的特例，当表中仅有一行数据时是 system。
12. null：执行式时不需要访问表或索引。如从索引列中取最大、最小值可以通过单独索引查找完成。explain select min(id) from test;

#### possible_keys （可能选择的索引）

#### key（实际选择的索引）

#### key_len（索引字段长度）

#### ref （列与索引比较，就是哪些列或常量被用于查找索引列上）

#### rows（扫描出的行数）

#### filtered（根据条件过滤后的百分比）

#### extra（其他参考信息）

> 尽量避免 using temporary、 using filesort

- using temporary：表示要用临时表来存储结果集。 explain select \* from test1 group by memid\G
- using filesort：当 query 中包含排序操作，且无法利用索引进行的排序操作称为 filesort。 explain select \* from test1 order by memid\G
- using where：全表扫描后，满足 where 条件的元素。 explain select \* from test1 where memid=1\G
- using index：当 query 使用了覆盖索引时，查询的字段和条件字段在二级索引就能完成，不回表。 explain select videoid from test1 where videoid=1
- using index condition：使用索引下推。
- using join buffer （Block Nested Loop）：在关联查询中，被驱动的表的关联字段如果没有索引就会用到 hash join。
- select tables optimized away：这意味着如果只使用索引，那么优化器可能仅从聚合函数中返回一行。 explain select min(videoid) from test2\G

### show profiles（获取 sql 执行时间）

> set profiling=1;
> 执行 sql 语句
> show profiles；

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671179753912-5c5ae8c0-0ae5-4632-9798-b2b613f05c2f.png#averageHue=%230d2a35&clientId=uc03c7f95-74b2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=263&id=u1220e921&margin=%5Bobject%20Object%5D&name=image.png&originHeight=526&originWidth=2654&originalType=binary∶=1&rotation=0&showTitle=false&size=146094&status=done&style=none&taskId=uf2e2312f-a9fd-425c-b11b-3623ed465e1&title=&width=1327)

### Performance Schma 分析

> 用于监控 Mysql 服务，并且运行时消耗很少的性能。
> Performance Schema 收集数据库服务器性能参数，并且表的存储引擎均为 Proformance_schema，而用户无法创建 performance_schema 表。

```bash
#查看是否开启 performance_schema
show variables like 'performance_schema';
#使用 performance_schema
use performance_schema;
# 修改 setup_instruments 状态，开启采集器
update performance_schema.setup_instruments set enabled = 'YES', timed = 'YES' where name like 'stage%';
#performance_schema 中的 setup_consumers 中保存着可用的消费者列表。
update performance_schema.setup_consumers set enabled='YES' where name like 'events%';
```

### Tace（追踪器）

> mysql 5.6 提供了对 sql 的追踪。
> 通过追踪进一步了解为什么优化器选择 A 执行计划而不是选择 B 执行计划，从而帮助用户更好的了解优化器行为。

```bash
#打开追踪器
set session optimizer_trace='enabled=on', end_markers_in_json=on;
#查看追踪器是否打开
show variables like ’optimizer_trace‘;
#查看追踪结果
select * from INFORMATION_SCHEMA.OPTIMIZER_TRACE；
```

```sql
Explain：针对SQL进行执行计划的评估，一般趋向于SQL本身性能的评估，评估结果可能和实际的执行结果不一样，有可能有误差。
Performance Schema：对SQL的每个阶段的执行过程进行时间评估，是实际的执行结果。
Trace追踪器：与Explain相比，Trace追踪器会对SQL的执行计划进行定量评估，让用户看到更详细的执行计划的评估过程，也是实际的执行结果。
```

\

## SQL 语句优化

1. 修改 sql 语句。
2. 加索引。
3. 改业务。

### 分页查询优化

> 当碰到较大的分页查询时，如 limit m，n ；m 的偏移量非常大时，需要扫描前 m 行数据，但是前 m 行数据不需要，所以查询效率低。

#### 利用子查询

```bash
#利用子查询，也就是利用覆盖索引方式先查询条件获取主键 id，然后根据主键 id 进行查询。

#第一步利用覆盖查询方式获取满足偏移量的主键 id。
select id from  test_log order by id limit 200000000,1;
#第二步使用第一步的结果进行分页查询。
select * from test_log where id>20000001 order by id limit 10;

#两步打包成一个 sql。

select * from test_log where id >=(select id from test_log order by id limit 2000000,1)
order by id limit 10;
```

> 覆盖索引是 select 的数据列只用从索引中就能够取得，不必读取数据行，换句话说查询列要被所建的[索引](https://baike.baidu.com/item/%E7%B4%A2%E5%BC%95/5716853?fromModule=lemma_inlink)覆盖。

大约能提升 50% 左右。

#### 利用延迟 join。

> 与方案一一样，也是利用子查询，先利用覆盖索引获取主键 id，在利用主键 id 查详细信息。

```sql
select * from test_log a join( select id from test_log order by id limit 20000000,10) b on a.id=b.id;
```

#### 书签方式

> 书签方式就是客户端记录上次查询结束的位置 id，下次分页这个 id 开始查。

### not in 优化

- 使用 left join
- 使用 not exists

优先级
left join>not exists>not in

```sql
#not in 语句
select * from test_log where content_id not in (select content_id from pre_log);

#使用 left join
select * from test_log a left join pre_log b on a.content_id=b.content_id where b.content_id is null;
#使用 not exists
select * from test_log where not exists (select content_id from pre_log);
```
