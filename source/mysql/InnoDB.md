>

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670980100621-0fe89948-adb7-424d-9be3-943f2aa6432e.png#averageHue=%23d7d7d7&clientId=udcc024a4-35dc-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=537&id=ue2fef0b3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1074&originWidth=1440&originalType=binary&ratio=1&rotation=0&showTitle=false&size=621393&status=done&style=none&taskId=u4b1032f3-6cba-4225-a94e-99a5705e3e3&title=&width=720)

## 内存结构

- `Buffer Pool`：缓冲池。InnoDB 在启动时分配的一个内存区域，用于 InnoDB 在访问数据时缓存表和索引数据。可以将 80%的物理内存分配给 InnoDB 缓冲池，为了提高缓存管理的效率，使用页面链表的方式+LRU（最近最少使用）算法进行管理。
- `Change Buffer`（Insert buffer part of buffer pool）:是一种特殊的数据结构（早期只支持 INSERT 操作的缓冲，所以也叫作 Insert Buffer），当受影响的页面不在缓冲池中时，将会缓存对辅助索引页的更改。这些更改可能是由 INSERT、UPDATE、DELETE（DML）语句执行时所导致的，当其他读取操作从磁盘中加载数据页时，如果这些数据页包含 Change Buffer 中缓存的更改操作页，那么将进行合并操作。
- `Adaptive Hash Index`：用于管理缓冲池中的内部数据结构。
- `Log Buffer`（Redo Log Buffer）：重做日志缓冲区。于保存将要写入重做日志磁盘文件中的数据的内存缓冲区域，重做日志缓冲区的大小由 innodb_log_buffer_size 配置参数定义。重做日志缓冲区中的内容会定期刷新到磁盘上的日志文件中。如果在应用场景中经常有大事务，则可以考虑增大重做日志缓冲区以减少磁盘 I/O 操作

## 磁盘结构

- System Tablespace：InnoDB 系统表空间包含 InnoDB 数据字典（InnoDB 相关对象的元数据）、Doublewrite Buffer、Change Buffer 磁盘部分和 Undo Logs，还包含在系统表空间中创建的任何表和索引数。
- Data Dictionary（InnoDB Data Dictionary）：InnoDB 数据字典由内部系统表组成，这些系统表包含用于跟踪对象（如表、索引和表列）的元数据。
- Doublewrite Buffer：双写缓冲区是一个位于系统表空间的存储区域，InnoDB 在进行刷脏操作时，在将脏数据写入数据文件中的正确位置之前先把脏页从 InnoDB 缓冲池写入双写缓冲区中。
- Undo Logs：用于存放事务修改之前的旧数据（undo log 记录了有关如何撤销事务对聚集索引记录的最新更改的信息），基于 undo 实现了 MVCC 和一致性非锁定读
- File-Per-Table Tablespaces：设置参数 innodb_file_per_table=1 启用独立表空间时，每个表都会对应生成一个．ibd 文件用于存放自己的索引和数据等；
- General Tablespaces：常规表空间
- Undo Tablespace:undo 表空间，包含一个或多个 undo log 文件
- Temporary Tablespace：临时表空间用于存放非压缩的 InnoDB 临时表和相关对象。
- Redo Logs：重做日志是在崩溃恢复期间使用的基于磁盘的数据结构文件，用于恢复不完整提交事务写入的数据。
