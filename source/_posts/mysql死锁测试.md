---
title: mysql 死锁测试
date: 2020-10-24 14:45:11
tags: mysql
---
#### 1. 创建测试表
```sql
CREATE TABLE `ty` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idxa` (`a`)
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8mb4;
```

#### 2. navicat 工具开启事务1窗口
```sql
BEGIN;                          //第一步，开启事务1
UPDATE ty SET b =1 WHERE a = 2; //第三步，锁定 a = 2 的记录
UPDATE ty SET b =1 WHERE a = 3; //第五步，卡主，等待事务2上 a = 3 的锁
COMMIT;                         
```

#### 3. navicat 工具开启事务2窗口
```sql
BEGIN;                          //第二步，开启事务2
UPDATE ty SET b =1 WHERE a = 3; //第四步，锁定 a = 3 的记录
UPDATE ty SET b =1 WHERE a = 2; //第六步，等待事务1上 a = 2 的锁，事务1和事务2互相等待对方的锁，此时出现死锁，mysql 会回退当前事务
COMMIT;                         
```
