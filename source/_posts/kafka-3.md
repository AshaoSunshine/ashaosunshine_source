---
title: kafka(三)
date: 2021-07-06 20:10:21
tags: Kafka
categories: 消息中间件
keywords: Kafka,kafka
top: 48
image: /images/kafka/kafka_tag_1.jpg
---

Kafka是一个分布式的基于发布/订阅的消息系统，有着强大的消息处理能力。
<!--more-->

# **Apache Kafka 是一个分布式流处理平台。**

## 1.0 Kafka HA 设计

### 1.1 将Replica均匀分布到整个集群

Kafka尽力把所有的Partition均匀分配到集群上是为了负载均衡。 一个典型的部署方式是一个Topic的Partition数量大于broker的数量。同时为了提高Kafka的容错能力，也需要将同一个Partition的Replica尽量分散到不同的机器。如果所有的Replica都在同一个Broker上，那一旦该Broker宕机，该Partition的所有Replica都无法工作，也就达不到HA的效果。同时，如果某个Broker宕机了，需要保证它上面的负载可以被均匀的分配到其它幸存的所有Broker上。

### 1.2 Data Replication(副本策略)

Kafka的高可靠性的保障来源于其健壮的副本（replication）策略。

#### 1.2.1 消息传递同步策略

producer在发布消息到某个Partition时，先通过Zookeeper找到该Partition的Leader，然后无论该Topic的Replication Factor为多少，Producer只将该消息发送到该Partition的Leader。Leader会将消息写入本地log。每个Follower都从Leader pull数据。这种方式上，Follower存储的数据顺序与Leader保持一致。

Kafka Replication的数据流如下图所示：

![](/images/kafka/kafka_3_1.png)

#### 1.2.2 选举Leader

最简单最直观的方案是，所有Follower都在ZooKeeper上设置一个Watch，一旦Leader宕机，其对应的ephemeral znode会自动删除，此时所有Follower都尝试创建该节点，而创建成功者（ZooKeeper保证只有一个能创建成功）即是新的Leader，其它Replica即为Follower。

但是该方法会有3个问题：

1.split-brain 这是由ZooKeeper的特性引起的，虽然ZooKeeper能保证所有Watch按顺序触发，但并不能保证同一时刻所有Replica“看”到的状态是一样的，这就可能造成不同Replica的响应不一致

2.herd effect 如果宕机的那个Broker上的Partition比较多，会造成多个Watch被触发，造成集群内大量的调整

3.ZooKeeper负载过重 每个Replica都要为此在ZooKeeper上注册一个Watch，当集群规模增加到几千个Partition时ZooKeeper负载会过重。

Kafka 0.8.*的Leader Election方案解决了上述问题，它在所有broker中选出一个controller，所有Partition的Leader选举都由controller决定。controller会将Leader的改变直接通过RPC的方式（比ZooKeeper Queue的方式更高效）通知需为为此作为响应的Broker。同时controller也负责增删Topic以及Replica的重新分配。

## 2.0 Kafka 高可用

### 2.1 Replication

在Kafka在0.8以前的版本中，是没有Replication的，一旦某一个Broker宕机，则其上所有的Partition数据都不可被消费，这与Kafka数据持久性及Delivery Guarantee的设计目标相悖。同时Producer都不能再将数据存于这些Partition中。

如果Producer使用同步模式则Producer会在尝试重新发送message.send.max.retries（默认值为3）次后抛出Exception，用户可以选择停止发送后续数据也可选择继续选择发送。而前者会造成数据的阻塞，后者会造成本应发往该Broker的数据的丢失。

如果Producer使用异步模式，则Producer会尝试重新发送message.send.max.retries（默认值为3）次后记录该异常并继续发送后续数据，这会造成数据丢失并且用户只能通过日志发现该问题。同时，Kafka的Producer并未对异步模式提供callback接口。

由此可见，在没有Replication的情况下，一旦某机器宕机或者某个Broker停止工作则会造成整个系统的可用性降低。随着集群规模的增加，整个集群中出现该类异常的几率大大增加，因此对于生产系统而言Replication机制的引入非常重要。

### 2.1.2 Leader Election

引入Replication之后，同一个Partition可能会有多个Replica，而这时需要在这些Replication之间选出一个Leader，Producer和Consumer只与这个Leader交互，其它Replica作为Follower从Leader中复制数据。

因为需要保证同一个Partition的多个Replica之间的数据一致性（其中一个宕机后其它Replica必须要能继续服务并且即不能造成数据重复也不能造成数据丢失）。如果没有一个Leader，所有Replica都可同时读/写数据，那就需要保证多个Replica之间互相（N×N条通路）同步数据，数据的一致性和有序性非常难保证，大大增加了Replication实现的复杂性，同时也增加了出现异常的几率。而引入Leader后，只有Leader负责数据读写，Follower只向Leader顺序Fetch数据（N条通路），系统更加简单且高效。

## 3.0 HA 相关ZooKeeper结构

![](/images/kafka/kafka_3_2.png)

### 3.1 admin

该目录下znode只有在有相关操作时才会存在，操作结束时会将其删除

/admin/reassign_partitions用于将一些Partition分配到不同的broker集合上。对于每个待重新分配的Partition，Kafka会在该znode上存储其所有的Replica和相应的Broker id。该znode由管理进程创建并且一旦重新分配成功它将会被自动移除。

### 3.2 broker

即/brokers/ids/[brokerId]）存储“活着”的broker信息。

topic注册信息（/brokers/topics/[topic]），存储该topic的所有partition的所有replica所在的broker id，第一个replica即为preferred replica，对一个给定的partition，它在同一个broker上最多只有一个replica,因此broker id可作为replica id。

### 3.3 controller

/controller -> int (broker id of the controller)存储当前controller的信息

/controller_epoch -> int (epoch)直接以整数形式存储controller epoch，而非像其它znode一样以JSON字符串形式存储。

