> 数据：/Users/mac/project/mysql/data
> 日志：/Users/mac/project/mysql/log/master

## 使用 docker 创建容器

```bash
#master1
docker run -p 3307:3306 --name mysql-master1 \
-v /Users/mac/project/mysql/master1/log:/var/log/mysql \
-v /Users/mac/project/mysql/master1/data:/var/lib/mysql \
-v /Users/mac/project/mysql/master1/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.27

#master2
docker run -p 3308:3306 --name mysql-master2 \
-v /Users/mac/project/mysql/master2/log:/var/log/mysql \
-v /Users/mac/project/mysql/master2/data:/var/lib/mysql \
-v /Users/mac/project/mysql/master2/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.27

#slave1
docker run -p 3309:3306 --name mysql-slave1 \
-v /Users/mac/project/mysql/slave1/log:/var/log/mysql \
-v /Users/mac/project/mysql/slave1/data:/var/lib/mysql \
-v /Users/mac/project/mysql/slave1/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.27


#slave2
docker run -p 3310:3306 --name mysql-slave2 \
-v /Users/mac/project/mysql/slave2/log:/var/log/mysql \
-v /Users/mac/project/mysql/slave2/data:/var/lib/mysql \
-v /Users/mac/project/mysql/slave2/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.27
```

### 创建配置

```bash
#master
[mysqld]

server-id=1001
expire_logs_days=7
max_binlog_size=1G

relay-log=myslql-relay-bin
log-bin=mysql-bin
#binlog 格式,取值：STATEMENT (默认)，ROW，MIXED
binlog_format=ROW



# 设置忽略的数据表
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
binlog-ignore-db=sys


#slave
[mysqld]

server-id=1004
relay-log=mysql-relay
```

### 创建同步账户

```sql
create user 'master1'@'%' identified with mysql_native_password by '123456';
grant replication slave,replication client on *.* TO 'master1'@'%';
flush privileges;

create user 'master2'@'%' identified with mysql_native_password by '123456';
grant replication slave,replication client on *.* TO 'master2'@'%';
flush privileges;
```

### 复制

slave1 复制 master1 slave2 复制 master2

```bash
#查看 ip 地址
docker inspect mysql-master1 |grep IPAddress
#查看 master 信息
show master status \G;

#同步master1
change master to master_host='172.17.0.2', \
master_user='master1', \
master_password='123456', \
master_log_file='mysql-bin.000002', \
master_log_pos=852;

#同步master2
change master to master_host='172.17.0.3', \
master_user='master2', \
master_password='123456', \
master_log_file='mysql-bin.000002', \
master_log_pos=852;
#执行同步
start slave;
#查看同步状态，看 lasterror 条数
show slave status\G;


#停止同步
stop slave

# 重置slave
reset slave;
#刷新
flush privileges



#两主机互相复制
master1复制master2, master2复制master1
同上
```
