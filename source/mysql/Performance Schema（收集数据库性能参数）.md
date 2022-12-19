> `用于收集数据库性能参数`
> Performance Schema 提供了有关 MySQL 服务器内部运行的操作上的底层指标
> 程序插桩（instrument）：在 mysql 中插入检测代码，从而获取到我们想了解的信息。
> 消费者表（consumer）：存储程序插桩代码的信息表。
> 启用程序插桩会调用额外的的代码，会消耗 CPU。
> 8.0 后 performance schema 有 110 个表。
> performance_schema 提供了一种在数据库运行时实时检查 Server 内部执行情况的方法
> 与 information_schema 不同， information_schema 主要关注 Server 运行过程中的元数据信息。

- performance_schema 通过监视 Server 的事件来实现监视其内部执行情况，“事件”就是在 Server 内部活动中所做的任何事情以及对应的时间消耗，利用这些信息来判断 Server 中的相关资源被消耗在哪里。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1669167914024-727c78f2-7714-4ea5-96c1-6c2105388778.png#averageHue=%23f5f5f5&clientId=u8b7b4b5e-aee7-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf556ca11&margin=%5Bobject%20Object%5D&name=image.png&originHeight=563&originWidth=1440&originalType=url&ratio=1&rotation=0&showTitle=false&size=188287&status=done&style=none&taskId=ubea11379-bb66-414c-8c95-862ad1a7349&title=)
查看是否开启

```bash
show variables like 'performance_schema';
//启用在 mysql 配置文件中
performance_schema = ON
```

## 表分类

### 配置表（setup）

```bash
use performance_schema;
show table like '%setup%';
+----------------------------------------+
| Tables_in_performance_schema (%setup%) |
+----------------------------------------+
| setup_actors                           |
| setup_consumers                        |
| setup_instruments                      |
| setup_objects                          |
| setup_threads                          |
+----------------------------------------+
```

- setup_actors：配置用户纬度的监控，默认监控所有用户。
- **setup_consumers：**配置 events 的消费者类型，即收集的 events 写入到哪些统计表中。
- **setup_instruments：**配置具体的 instrument，主要包含 4 大类：idle、stage/xxx、statement/xxx、wait/xxx：
- **setup_objects：**配置监控对象，默认对 mysql，performance_schema 和 information_schema 中的表都不监控，而其它 DB 的所有表都监控。
- **setup_timers：**配置每种类型指令的统计时间单位。

### 实例表（**instance**）

- **ond_instances**：条件等待对象实例
- **file_instances**：文件实例
- **mutex_instances：**互斥同步对象实例
- **rwlock_instances：** 读写锁同步对象实例
- **socket_instances：**活跃会话对象实例

### 等待表（wait）

- **events_waits_current**：记录了当前线程等待的事件。
- **events_waits_history**：记录了每个线程最近等待的 10 个事件。
- **events_waits_history_long**：记录了最近所有线程产生的 10000 个事件。

### 执行阶段表（stage）

- **events_stages_current**：记录了当前线程所处的执行阶段。
- **events_stages_history**：记录了当前线程所处的执行阶段 10 条历史记录。
- **events_stages_history_long**：记录了当前线程所处的执行阶段 10000 条历史记录。

### 语句表（**Statement**）

- **events_statements_current**：通过 thread_id+event_id 可以唯一确定一条记录。Statments 表只记录最顶层的请求，SQL 语句或是 COMMAND，每条语句一行。event_name 形式为 statement/sql/_，或 statement/com/_
- **events_statements_history**
- **events_statements_history_long**

### 连接表（**Connection**）

- **users**：记录用户连接数信息。
- **hosts**：记录了主机连接数信息。
- **accounts**：记录了用户主机连接数信息。

### 概要表（**Summary**）

> **Summary 表聚集了各个维度的统计信息包括表维度，索引维度，会话维度，语句维度和锁维度的统计信息**

- **events_waits_summary_global_by_event_name**：按等待事件类型聚合，每个事件一条记录。
- **events_waits_summary_by_instance**：按等待事件对象聚合，同一种等待事件，可能有多个实例，每个实例有不同的内存地址，因此 event_name+object_instance_begin 唯一确定一条记录.
- **events_waits_summary_by_thread_by_event_name**：按每个线程和事件来统计，thread_id+event_name 唯一确定一条记录。
- **events_stages_summary_global_by_event_name**：按事件阶段类型聚合，每个事件一条记录，表结构同上。
- **events_stages_summary_by_thread_by_event_name**：按每个线程和事件来阶段统计，表结构同上.
- **events_statements_summary_by_digest**：按照事件的语句进行聚合。
- **events_statements_summary_global_by_event_name**：按照事件的语句进行聚合。表结构同上.
- **events_statements_summary_by_thread_by_event_name**：按照线程和事件的语句进行聚合，表结构同上。
- **file_summary_by_instance**：按事件类型统计（**物理 IO 维度**）
- **file_summary_by_event_name**：具体文件统计（**物理 IO 维度**）
- **socket_summary_by_instance、socket_summary_by_event_name**：socket 聚合统计表。

```sql
CREATE TABLE `events_waits_summary_global_by_event_name` (
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8


CREATE TABLE `events_waits_summary_by_instance` (
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `OBJECT_INSTANCE_BEGIN` bigint(20) unsigned NOT NULL COMMENT '内存地址',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8

CREATE TABLE `events_waits_summary_by_thread_by_event_name` (
  `THREAD_ID` bigint(20) unsigned NOT NULL COMMENT '线程ID',
  `EVENT_NAME` varchar(128) NOT NULL COMMENT '事件名称',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8


CREATE TABLE `events_statements_summary_by_digest` (
  `SCHEMA_NAME` varchar(64) DEFAULT NULL COMMENT '库名',
  `DIGEST` varchar(32) DEFAULT NULL COMMENT '对SQL_TEXT做MD5产生的32位字符串。如果为consumer表中没有打开statement_digest选项，则为NULL',
  `DIGEST_TEXT` longtext COMMENT '将语句中值部分用问号代替，用于SQL语句归类。如果为consumer表中没有打开statement_digest选项，则为NULL。',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间',
  `SUM_LOCK_TIME` bigint(20) unsigned NOT NULL COMMENT '锁时间总时长',
  `SUM_ERRORS` bigint(20) unsigned NOT NULL COMMENT '错误数的总',
  `SUM_WARNINGS` bigint(20) unsigned NOT NULL COMMENT '警告的总数',
  `SUM_ROWS_AFFECTED` bigint(20) unsigned NOT NULL COMMENT '影响的总数目',
  `SUM_ROWS_SENT` bigint(20) unsigned NOT NULL COMMENT '返回总数目',
  `SUM_ROWS_EXAMINED` bigint(20) unsigned NOT NULL COMMENT '总的扫描的数目',
  `SUM_CREATED_TMP_DISK_TABLES` bigint(20) unsigned NOT NULL COMMENT '创建磁盘临时表的总数目',
  `SUM_CREATED_TMP_TABLES` bigint(20) unsigned NOT NULL COMMENT '创建临时表的总数目',
  `SUM_SELECT_FULL_JOIN` bigint(20) unsigned NOT NULL COMMENT '第一个表全表扫描的总数目',
  `SUM_SELECT_FULL_RANGE_JOIN` bigint(20) unsigned NOT NULL COMMENT '总的采用range方式扫描的数目',
  `SUM_SELECT_RANGE` bigint(20) unsigned NOT NULL COMMENT '第一个表采用range方式扫描的总数目',
  `SUM_SELECT_RANGE_CHECK` bigint(20) unsigned NOT NULL COMMENT '',
  `SUM_SELECT_SCAN` bigint(20) unsigned NOT NULL COMMENT '第一个表位全表扫描的总数目',
  `SUM_SORT_MERGE_PASSES` bigint(20) unsigned NOT NULL COMMENT '',
  `SUM_SORT_RANGE` bigint(20) unsigned NOT NULL COMMENT '范围排序总数',
  `SUM_SORT_ROWS` bigint(20) unsigned NOT NULL COMMENT '排序的记录总数目',
  `SUM_SORT_SCAN` bigint(20) unsigned NOT NULL COMMENT '第一个表排序扫描总数目',
  `SUM_NO_INDEX_USED` bigint(20) unsigned NOT NULL COMMENT '没有使用索引总数',
  `SUM_NO_GOOD_INDEX_USED` bigint(20) unsigned NOT NULL COMMENT '',
  `FIRST_SEEN` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '第一次执行时间',
  `LAST_SEEN` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '最后一次执行时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8

```

### 其他相关表

1. **performance_timers**：系统支持的统计时间单位。
2. **threads**：监视服务端的当前运行的线程。

### 统计应用

> ** 关于 SQL 维度的统计信息主要集中在 events_statements_summary_by_digest 表中，通过将 SQL 语句抽象出 digest，可以统计某类 SQL 语句在各个维度的统计信息**

**表结构：**

```sql
CREATE TABLE `events_statements_summary_by_digest` (
  `SCHEMA_NAME` varchar(64) DEFAULT NULL COMMENT '库名',
  `DIGEST` varchar(32) DEFAULT NULL COMMENT '对SQL_TEXT做MD5产生的32位字符串。如果为consumer表中没有打开statement_digest选项，则为NULL',
  `DIGEST_TEXT` longtext COMMENT '将语句中值部分用问号代替，用于SQL语句归类。如果为consumer表中没有打开statement_digest选项，则为NULL。',
  `COUNT_STAR` bigint(20) unsigned NOT NULL COMMENT '事件计数',
  `SUM_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '总的等待时间',
  `MIN_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最小等待时间',
  `AVG_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '平均等待时间',
  `MAX_TIMER_WAIT` bigint(20) unsigned NOT NULL COMMENT '最大等待时间',
  `SUM_LOCK_TIME` bigint(20) unsigned NOT NULL COMMENT '锁时间总时长',
  `SUM_ERRORS` bigint(20) unsigned NOT NULL COMMENT '错误数的总',
  `SUM_WARNINGS` bigint(20) unsigned NOT NULL COMMENT '警告的总数',
  `SUM_ROWS_AFFECTED` bigint(20) unsigned NOT NULL COMMENT '影响的总数目',
  `SUM_ROWS_SENT` bigint(20) unsigned NOT NULL COMMENT '返回总数目',
  `SUM_ROWS_EXAMINED` bigint(20) unsigned NOT NULL COMMENT '总的扫描的数目',
  `SUM_CREATED_TMP_DISK_TABLES` bigint(20) unsigned NOT NULL COMMENT '创建磁盘临时表的总数目',
  `SUM_CREATED_TMP_TABLES` bigint(20) unsigned NOT NULL COMMENT '创建临时表的总数目',
  `SUM_SELECT_FULL_JOIN` bigint(20) unsigned NOT NULL COMMENT '第一个表全表扫描的总数目',
  `SUM_SELECT_FULL_RANGE_JOIN` bigint(20) unsigned NOT NULL COMMENT '总的采用range方式扫描的数目',
  `SUM_SELECT_RANGE` bigint(20) unsigned NOT NULL COMMENT '第一个表采用range方式扫描的总数目',
  `SUM_SELECT_RANGE_CHECK` bigint(20) unsigned NOT NULL COMMENT '',
  `SUM_SELECT_SCAN` bigint(20) unsigned NOT NULL COMMENT '第一个表位全表扫描的总数目',
  `SUM_SORT_MERGE_PASSES` bigint(20) unsigned NOT NULL COMMENT '',
  `SUM_SORT_RANGE` bigint(20) unsigned NOT NULL COMMENT '范围排序总数',
  `SUM_SORT_ROWS` bigint(20) unsigned NOT NULL COMMENT '排序的记录总数目',
  `SUM_SORT_SCAN` bigint(20) unsigned NOT NULL COMMENT '第一个表排序扫描总数目',
  `SUM_NO_INDEX_USED` bigint(20) unsigned NOT NULL COMMENT '没有使用索引总数',
  `SUM_NO_GOOD_INDEX_USED` bigint(20) unsigned NOT NULL COMMENT '',
  `FIRST_SEEN` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '第一次执行时间',
  `LAST_SEEN` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '最后一次执行时间'
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8
```

#### 哪个 sql 执行最多（COUNT_STAR）

```sql
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by COUNT_STAR desc limit 1\G
```

#### **哪个 SQL 平均响应时间最多（**AVG_TIMER_WAIT**）**

```sql
SELECT SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN FROM events_statements_summary_by_digest ORDER BY AVG_TIMER_WAIT desc LIMIT 1\G
```

#### **哪个 SQL 扫描的行数最多（**SUM_ROWS_EXAMINED**）**

```sql
SELECT SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN FROM events_statements_summary_by_digest ORDER BY SUM_ROWS_EXAMINED desc LIMIT 1\G
```

#### **哪个 SQL 扫描的行数最多（**SUM_ROWS_EXAMINED**）**

```sql
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by SUM_ROWS_EXAMINED desc limit 1\G

```

#### **哪个 SQL 使用的临时表最多（**SUM_CREATED_TMP_DISK_TABLES、SUM_CREATED_TMP_TABLES**）**

#### **哪个 SQL 返回的结果集最多（**SUM_ROWS_SENT**）**

#### **哪个 SQL 排序数最多（**SUM_SORT_ROWS**）**

#### **哪个表、文件逻辑 IO 最多（热数据）**

```sql
SELECT FILE_NAME,EVENT_NAME,COUNT_READ,SUM_NUMBER_OF_BYTES_READ,COUNT_WRITE,SUM_NUMBER_OF_BYTES_WRITE FROM file_summary_by_instance ORDER BY SUM_NUMBER_OF_BYTES_READ+SUM_NUMBER_OF_BYTES_WRITE DESC LIMIT 2\G
```

#### **哪个索引使用最多（**table_io_waits_summary_by_index_usage，SUM_TIMER_WAIT**）**

```sql
SELECT OBJECT_NAME, INDEX_NAME, COUNT_FETCH, COUNT_INSERT, COUNT_UPDATE, COUNT_DELETE FROM table_io_waits_summary_by_index_usage ORDER BY SUM_TIMER_WAIT DESC limit 1;
```

#### **哪个索引没有使用过（**table_io_waits_summary_by_index_usage**）**

```sql
SELECT OBJECT_SCHEMA, OBJECT_NAME, INDEX_NAME FROM table_io_waits_summary_by_index_usage WHERE INDEX_NAME IS NOT NULL AND COUNT_STAR = 0 AND OBJECT_SCHEMA <> 'mysql' ORDER BY OBJECT_SCHEMA,OBJECT_NAME;
```

#### **哪个等待事件消耗的时间最多（**events_waits_summary_global_by_event_name**）**

```sql
SELECT EVENT_NAME, COUNT_STAR, SUM_TIMER_WAIT, AVG_TIMER_WAIT FROM events_waits_summary_global_by_event_name WHERE event_name != 'idle' ORDER BY SUM_TIMER_WAIT DESC LIMIT 1;
```

####

## 插桩元件

> setup_instruments 表包含所有支持插桩的列表，可以查看该表，修改该表中的 enable='YES' , timed='YES' 表示开始记录分析。
> 插桩名称（NAME 列）的最左边部分表示插桩的类型。

## 消费者表的组织

### 当前和历史数据

- \*\_current：当前服务器进行中的事件。
- \*\_history：每个线程最近完成的 10 个事件。
- \*\_history_long：每个线程最近完成的 10000 个事件。
- events_waits：底层服务器等待，如获取互斥对象。
- **events_statments：sql 查询语句。**
- events_stages：配置文件信息，如创建临时表时或发送数据。
- events_transactions：事务。

### 汇总表和摘要

> 汇总表中保存有该表所建议的内容的聚合信息。
> memory_summary_by_thread_by_event_name 表存了用户连接或任何后台线程的每个 MySQL 线程的聚合内存使用情况。

### 性能消耗

> 与 statement 相关的插桩在查询过程中只能被调用一次，而 wait 类插桩的被调用频率要高得多。例如，要扫描一个有一百万行的 InnoDB 表，引擎需要设置和释放一百万行锁。如果对锁使用 wait 类插桩，CPU 使用率可能会显著增加。
> Performance Schema 收集的数据保存在内存中。可以通过设置消费者表的最大大小来限制其使用的内存量。

### 线程

> MySQL 服务端是多线程软件。它的每个组件都使用线程。
> 由主线程或存储引擎创建的，也可以是为用户连接创建的前台线程。
> 个线程至少有两个唯一标识符：一个是操作系统线程 ID，另一个是 MySQL 内部线程 ID。
> 而 MySQL 内部线程 ID 在大多数 performance_schema 表中以 THREAD_ID 命名。
> 在 Linux 系统中可使用`ps-eLf`命令查看。
> 连接标识符，在 SHOW PROCESSLIST 命令输出中或在 MySQL 命令行客户端连接时在“Your MySQLconnection idis”字符串中可以看到。

### 启用禁用插桩

- 方法一：update 语句， UPDATE performance_schema.setup_instruments SET ENABLED='YES' WHERE NAME='statement/sql/select';
- 方法二：sys schema 提供了两个存储过程——ps_setup_enable_instrument 和 ps_setup_disable_instrument——用于启用和禁用其参数所对应的插桩。

### 使用
