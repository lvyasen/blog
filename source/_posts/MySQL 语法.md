---
title: MySQL 语法
urlname: cyxh786r7pm1t1vq
date: '2022-11-18 21:14:20 +0800'
tags:
  - mysql
categories:
  - mysql
---

## 查询语法

> 一般 sql 以分号结束，不区分大小写，多行可以换行

```sql
#查看版本
show version();

#查询用户
select user();

#查询数据列表
show databases;

#使用数据库
use mydatabase;

#查看数据表
show tables;

#创建数据表
create table user (name varchar(20), age int(3), nickname varchar(60), sex tinyint(3));

#查看数据表详情
describe user;

#将文本写入到数据表中
LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet;

#插入数据
insert into pet value ('Puffball','Diane','hamster','f','1999-03-30',NULL);

#查看表
select * from pet;

#删除表
delete from pet;

#查询唯一记录
select distinct name from pet;

#排序
select * from pet order by birth;

#排序强制区分大小写
select * from pet order by binary birth;

#倒序
select * from pet order by birth desc;

#多个字段排序
select * from pet order by owner asc , birth desc;

#日期计算

```
