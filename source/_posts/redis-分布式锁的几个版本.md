---
title: redis 分布式锁的几个版本
date: 2020-11-17 08:49:27
tags: [redis,分布式,锁]
categories: redis
---

## 1、版本一

原理：setKey 加锁；deleteKey 解锁。
问题：加锁的服务挂掉的话，锁无法释放，其他服务无法获取锁。

## 2、版本二

原理：setKey 加过期时间加锁；deleteKey 解锁。
问题：A服务获取了锁，但是业务操作时间过长，超过了 redis key 的时长，那么其他服务就会获取同一把锁。

## 3、版本三

原理：setKey + 线程ID 来加锁，同时起一个定时任务，当业务操作时间过长为 key 续命；解锁时判断 线程ID 是否来自同一个服务，是就解锁。
问题：解锁过程包含了取值、判断、删除key三个步骤，不是原子性会有并发问题。

## 4、版本四

原理：setKey + 线程ID 来加锁，同时起一个定时任务，当业务操作时间过长为 key 续命；解锁时用 lua 脚本的原子性来解锁，解决并发问题。
问题：redis 集群下不能用。

综上可见，redis 锁没有百分百的完美，但是版本四基本可以满足日常的使用需求了。

**版本四 Spring 提供了实现方案：[spring-integration-redis](https://docs.spring.io/spring-integration/reference/html/redis.html)**