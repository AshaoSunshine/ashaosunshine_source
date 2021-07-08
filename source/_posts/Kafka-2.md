---
title: Kafka(二)
date: 2021-07-04 20:58:15
tags: Kafka
categories: 技术
keywords: Kafka,kafka
top: 49
---

Kafka是一个分布式的基于发布/订阅的消息系统，有着强大的消息处理能力。
<!--more-->

# **Apache Kafka 是一个分布式流处理平台。**

## 1.0 Kafka结构

Kafka集群包含若干Producer，若干broker（broker数量越多，集群吞吐率越高），若干Consumer Group，以及一个Zookeeper集群。Kafka通过Zookeeper管理集群配置，选举leader，以及在Consumer Group发生变化时进行rebalance。Producer使用push模式将信息发布到broker，Consumer使用pull模式从broker订阅并消费消息。

![](/images/kafka/kafka_02.png)

## 2.0 Producer消息

Producer发送消息到broker时，会根据Partition机制选择将其存储到哪一个Partition。如果Partition机制设置合理，所有消息可以均匀分布到不同的Partition里，这样就视线里负载均衡。如果一个Topic对应一个文件，那这个文件所在的机器I/O就会变成这个Topic的性能瓶颈，但有了Partition后，不同的消息可以并行写入不同broker的不同的Partition里，可以极大地提高吞吐率。

## 3.0 Topics和Partition

Topic在逻辑上可以被认为是一个queue，每条消费都必须指定他的Topic，可以简单理解为必须指名把这条消息放进哪个queue里。为了使得Kafka的吞吐率可以线性提高，物理上把Topic分成一个或多个Partition在物理上对应一个文件夹，该文件夹下存储这个Partition的所有消息和索引文件。创建一个topic时，同时可以指定分区数目，分区数越多，其吞吐量也越大，但是需要的资源也越多，同时会导致更高的不可用性，kafka在接收到生产者发送的消息之后，会根据均衡策略将消息存储到不同的分区中。（顺序写磁盘效率比随机写内存还要高，这是Kafka高吞吐率的一个很重要的保证）。

## 4.0 Consumer Group

![](/images/kafka/consumer_groups.png)

使用Consumer high level API时，同一Topic的一条消息只能被同一个Consumer Group内的一个Consumer消费，但多个Consumer Group 可同时消费这一消息。

这是Kafka用来实现一个Topic消息的广播（发给所有的Consumer）和单播（发给某个Consumer）的手段。一个Topic可以对应多个Consumer Group。如果需要广播，只要每个Consumer有一个独立的Group就可以了。要实现单播只要所有的Comsumer在同一个Group里。用Consumer Group还可以将Consumer进行自由的分组而不是需要多次发送消息到不同的Topic.


## 5.0 push and pull

Kafka作为一个消息系统遵循了传统的方式，选择又Producer向broker push消息并由Consumer从broker pull消息。

push模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。push模式的目标是尽可能以最快速度传递消息，但是这样很容易造成Consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据Consumer的消费能力以适当的速率消费消息。

Kafka更适合pull模式，pull模式可简化broker的设计，Consumer可自主控制消费消息的速率，同时Consumer可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。