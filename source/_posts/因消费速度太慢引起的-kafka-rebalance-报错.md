---
title: 因消费速度太慢引起的 kafka rebalance 报错
date: 2020-11-17 09:06:50
tags: kafka
---
之前项目使用 kafka 有个 rebalance 报错的问题，折腾了好一会，后来搞明白是怎么回事，记录一下。
## 1、报错信息

```java
08-09 11:01:11 131 pool-7-thread-3 ERROR [] - 
commit failed 
org.apache.kafka.clients.consumer.CommitFailedException: Commit cannot be completed since the group has already rebalanced and assigned the partitions to another member. This means that the time between subsequent calls to poll() was longer than the configured max.poll.interval.ms, which typically implies that the poll loop is spending too much time message processing. You can address this either by increasing the session timeout or by reducing the maximum size of batches returned in poll() with max.poll.records.
        at org.apache.kafka.clients.consumer.internals.ConsumerCoordinator.sendOffsetCommitRequest(ConsumerCoordinator.java:713) ~[MsgAgent-jar-with-dependencies.jar:na]
        at org.apache.kafka.clients.consumer.internals.ConsumerCoordinator.commitOffsetsSync(ConsumerCoordinator.java:596) ~[MsgAgent-jar-with-dependencies.jar:na]
        at org.apache.kafka.clients.consumer.KafkaConsumer.commitSync(KafkaConsumer.java:1218) ~[MsgAgent-jar-with-dependencies.jar:na]
        at com.today.eventbus.common.MsgConsumer.run(MsgConsumer.java:121) ~[MsgAgent-jar-with-dependencies.jar:na]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_161]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_161]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_161]
```
## 2、报错原因
这个报错是消费者在处理完上次 poll 的消息后，在同步提交偏移量给 broker 时报的错。初步分析日志是由于当前消费者线程消费的分区已经被 broker 给回收了，因为kafka认为这个消费者死了。
消费者在创建时有一个重要的配置`max.poll.interval.ms`，这个配置的意思是kafka消费者在每次 poll() 调用之间的最大延迟，消费者在获取更多记录之前可以空闲的时间量的上限。如果此超时时间期满之前 poll() 没有被再次调用，则消费者被视为失败，并且分组将重新平衡，以便将分区重新分配给别的成员。
**那么简单来说就是，消费者消费速度跟不上，无法在指定时间内消费完上次 poll 的数据。**
## 3、解决办法
解决方法很简单，根据自身消费速度调整配置。
1、调大 max.poll.interval.ms 
2、调小 max.poll.records
**我最终选择了第二个办法，除此以外我还 review 了消费的业务代码，发现了一条慢 sql 拖垮了消费速度，所以最大的问题实质是慢 sql，优化后，完美解决这个问题。**