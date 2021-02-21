---
title: mysql 重置主从复制
date: 2020-09-26 14:15:39
tags: [主从配置,教程,步骤]
categories: mysql
---
数据库主从在实际使用之中，竟然由于各种各样的原因导致数据不一致，需要重新配置。
# 1、连接主库
```sql
SHOW MASTER STATUS；
```
记录下 master_log_pos 的位置，后面需要用到。
# 2、连接从库
## 2.1、停用从库
```sql
STOP SLAVE;
```
## 2.2、重置从库
```sql
reset slave;
```
## 2.3、修改主从指向配置
```sql
CHANGE MASTER TO master_host = '172.20.5.20',
 master_user = 'root',
 master_password = 'rootroot',
 master_port = 3306,
 master_log_file = 'edu-mysql-bin.000016',
 master_log_pos = 978213840,
 master_connect_retry = 30;
```
## 2.4、开启同步
```sql
START SLAVE;
```
另外，测试环境也经常遇到由于表结构修改后不同步，导致数据同步出错，此时可以跳过错误的语句使得测试进行下去。
```sql
stop slave; 
set global sql_slave_skip_counter = 1;
start slave;
```