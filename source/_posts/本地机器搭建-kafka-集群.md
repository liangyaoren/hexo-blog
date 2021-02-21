---
title: 本地机器搭建 kafka 集群
date: 2020-10-26 10:34:18
tags: [教程,步骤]
categories: kafka
---
# 1、虚拟三台CentOS
1、管理员模式启动 VMware。
2、VMware => 编辑 => 虚拟网络编辑器 =>选择 Net 模式；关闭 dhcp；记录下网关地址。
3、虚拟出三台主机后，修改静态ip，网段和第2点的网段一样，网关对应第2点的网关地址。重启 service network restart。
4、修改宿主主机的 VMware Network Adapter VMnet8 的 ip 地址，设置和第2点同样的网段地址。（非常重要，否则会出现主机 ping 不通虚拟机的情况）
参考文章 https://www.linuxidc.com/Linux/2017-05/144328.htm
# 2、安装 zookeeper
1、参考文章
https://www.cnblogs.com/luotianshuai/p/5206662.html

2、替换可用yum源，非常重要，参考其他笔记

3、安装jdk
```shell
yum -y install java-1.8.0-openjdk*
```

4、zookeeper 安装简要命令
```shell
mkdir /home/zookeeper
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/stable/apache-zookeeper-3.5.5-bin.tar.gz
tar -xvf /home/zookeeper/apache-zookeeper-3.5.5-bin.tar.gz
cd /home/zookeeper
mkdir zkdata
mkdir zkdatalog
cp apache-zookeeper-3.5.5-bin/conf/zoo_sample.cfg apache-zookeeper-3.5.5-bin/conf/zoo.cfg
vi apache-zookeeper-3.5.5-bin/conf/zoo.cfg
//配置下面内容-----
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/zookeeper/zkdata
dataLogDir=/home/zookeeper/zkdatalog
clientPort=2181
server.1=192.168.75.86:2888:3888
server.2=192.168.75.87:2888:3888
server.3=192.168.75.88:2888:3888
//配置内容结束------
```
5、启动 zookeeper
```shell
/home/zookeeper/apache-zookeeper-3.5.5-bin/bin/zkServer.sh start //直接启动
/home/zookeeper/apache-zookeeper-3.5.5-bin/bin/zkServer.sh start-foreground //边启动边看日志
```
6、查看状态
```shell
/home/zookeeper/apache-zookeeper-3.5.5-bin/bin/zkServer.sh status
```
# 3、安装 kafka 
1、kafka 安装简要命令
```shell
mkdir /home/kafka
wget ...
tar -xvf ...
vi server.properties
配置下面两项---
host.name=192.168.75.86
zookeeper.connect=192.168.75.86:2181,192.168.75.87:2181,192.168.75.88:2181
结束配置-----
```
2、启动
```shell
/home/kafka/kafka_2.12-2.2.0/bin/kafka-server-start.sh -daemon /home/kafka/kafka_2.12-2.2.0/config/server.properties
```
3、查看日志
```shell
tail -fn 200 /home/kafka/kafka_2.12-2.2.0/logs/server.log 
```
4、创建主题
```shell
./kafka-topics.sh --create --zookeeper 192.168.2.121:2181 --replication-factor 3 --partitions 10 --topic request-online
```