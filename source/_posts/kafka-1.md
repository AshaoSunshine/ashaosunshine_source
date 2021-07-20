---
title: Kafka(一)
date: 2021-07-01 16:38:09
tags: Kafka
categories: 消息中间件
keywords: Kafka,kafka
top: 50
image: /images/kafka/kafka_tag_1.jpg
---

Kafka是一个分布式的基于发布/订阅的消息系统，有着强大的消息处理能力。
<!--more-->

# **Apache Kafka 是一个分布式流处理平台。**

## 1.0 介绍

kafka 作为一个集群，运行在一台或者多台服务器上，它是通过topic对存储的流数据进行分类。每条记录中包含一个key,一个value和一个timestamp(时间戳)。

Kafka是Apache下的一个子项目，是一个高性能跨语言分布式发布/订阅消息队列系统，而Jafka是在Kafka之上孵化而来的，即Kafka的一个升级版。具有以下特性：快速持久化，可以在O(1)的系统开销下进行消息持久化；高吞吐，在一台普通的服务器上既可以达到10W/s的吞吐速率；完全的分布式系统，Broker、Producer、Consumer都原生自动支持分布式，自动实现负载均衡；支持Hadoop数据并行加载，对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka通过Hadoop的并行加载机制统一了在线和离线的消息处理。Apache Kafka相对于ActiveMQ是一个非常轻量级的消息系统，除了性能非常好之外，还是一个工作良好的分布式系统。

kafka有四个核心的API：
- Producer API允许一个应用程序发布一串流式的数据到一个或者多个Kafka topic。
- Consumer API允许一个应用程序订阅一个或者多个topic,并且对发布给他们的流式数据进行处理。
- Streams API允许一个应用程序作为一个流处理器，消费一个或者多个topic产生的输入流，然后产生一个输出流到一个或者多个topic中去，在输入输出流中进行有效的转换。
- Connector API允许构建并运行可重用的生产者或者消费者，将Kafka Topic连接到已经存在的应用程序或者数据系统。
![](/images/kafka/kafka-apis.png)

### 1.1 相关术语

#### 1.1.1 broker

Kafka 集群包含一个或多个服务器，服务器节点称为broker。

broker存储topic的数据。如果某topic有N个partition，集群有N个broker，那么每个broker存储该topic的一个partition。

如果某topic有N个partition，集群有(N+M)个broker，那么其中有N个broker存储该topic的一个partition，剩下的M个broker不存储该topic的partition数据。

如果某topic有N个partition，集群中broker数目少于N个，那么一个broker存储该topic的一个或多个partition。在实际生产环境中，尽量避免这种情况的发生，这种情况容易导致Kafka集群数据不均衡。

#### 1.1.2 Topic

每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）

类似于数据库的表名

#### 1.1.3 Partition

topic中的数据分割为一个或多个partition。每个topic至少有一个partition。每个partition中的数据使用多个segment文件存储。partition中的数据是有序的，不同partition间的数据丢失了数据的顺序。如果topic有多个partition，消费数据时就不能保证数据的顺序。在需要严格保证消息的消费顺序的场景下，需要将partition数目设为1。

#### 1.1.4 Producer

生产者即数据的发布者，该角色将消息发布到Kafka的topic中。broker接收到生产者发送的消息后，broker将该消息追加到当前用于追加数据的segment文件中。生产者发送的消息，存储到一个partition中，生产者也可以指定数据存储的partition。

#### 1.1.5 Consumer

消费者可以从broker中读取数据。消费者可以消费多个topic中的数据。

#### 1.1.6 Consumer Group

每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。

#### 1.1.7 Leader

每个partition有多个副本，其中有且仅有一个作为Leader，Leader是当前负责数据的读写的partition。

#### 1.1.8 Follower

Follower跟随Leader，所有写请求都通过Leader路由，数据变更会广播给所有Follower，Follower与Leader保持数据同步。如果Leader失效，则从Follower中选举出一个新的Leader。当Follower与Leader挂掉、卡住或者同步太慢，leader会把这个follower从“in sync replicas”（ISR）列表中删除，重新创建一个Follower。

### 1.2 Topics和日志
Topic 是数据主题，是数据记录发布的地方，可以用来区分业务系统。Kafka中的Topics总是多订阅者模式，一个topic可以拥有一个或者多个消费之来订阅它的数据。

每一个topic，Kafka集群会维持一个分区日志。

![](/images/kafka/log_anatomy.png)

每个分区都是有序且顺序不可变的记录集，并且不断地追加到结构化的commit log文件。分区中的每一个记录都会分配一个id来表示顺序，我们称为offset,offset用来唯一的标识分区中的每一条数据。

kafka的性能和数据大小无关，所以长时间存储数据没什么问题。

日志中的partition分区有以下几个用途。第一，当日志大小超过了单台服务器的限制，允许日志进行扩展。每个单独的分区都必须受限于主机的文件限制，不过一个主题可以有多个分区，因此可以处理无限量的数据。第二，可以作为并行的单元集。

### 1.3 分布式

日志的分区partition分布在Kafka集群的服务器上。每个服务器在处理数据和请求时，共享这些分区。每一个分区都会在已配置的服务器上进行备份，确保容错性。

每个分区都有一台server作为“leader”， 0台或者多台server作为follers.leader server处理一切对partition分区的读写请求，而follwers只需被动的同步leader上的数据。当leader宕机了，followers中的一台服务器会自动成为新的leader。每台server都会成为某些分区的leader和某些分区的follower，因此集群的负载是平衡的。

### 1.4 生产者

生产者可以将数据发布到所选择的topic(主题)中。生产者负责记录分配到topic中的某个partition分区中。可以使用循环的方式来简单地实现负载均衡。

### 1.5 消费者

消费者使用一个消费组名称来进行标识，发布到topic中的每条记录被分配给订阅消费组中的消费者实例。消费者实例可以分布在多个进程中或者多个机器上。

如果所有的消费者实例在同一消费组中，消费记录会负载均衡到每一个消费者实例。

如果所有的消费者实例在不同的消费组中，每条消息记录会广播到所有的消费者进程。
![](/images/kafka/consumer_groups.png)

如图，这个Kafka集群有两台server，四个分区和两个消费者组。消费组A有两个消费者，消费组B有四个消费者。

通常情况下，每个topic都会有一些消费组，一个消费组对应一个“逻辑订阅者”。 一个消费组由许多消费者实例组成，便于扩展和容错。这就是发布和订阅的概念，只不过订阅者是一组消费者而不是单个进程。

在Kafka中实现消费的方式是将日志中的分区划分到每一个消费者实例上，以便在任何时间，每个实例都是分区唯一的消费者。维护消费者组中的消费关系由Kafka协议动态处理。如果新的实例加入组，他们将从组中其他成员处接管一些partition分区；如果一个实例消失，拥有的分区将被分发至剩余的实例。

Kafka只保证分区内记录是有序的，不保证主题中不同分区的顺序。

### 1.6 Kafka作为存储系统

数据写入Kafka后被写入到磁盘，并且备份以便容错，直到完全备份，Kafka才让生产者认为完成写入，即使写入失败kafka也会确保继续写入Kafka使用磁盘结构，具有很好的扩展性。

Kafka是一种高性能、低延迟、具备日志存储、备份和传播功能的分布式文件系统。

### 1.7 Kafka用作流处理

Kafka流处理不仅用来读写和存储流式数据，最终目的是为了能够进行实时的流处理。在Kafka中，流处理不断地从输入的topic获取流数据，处理数据后，在不断生产数据到输出的topic中去。

简单数据处理可以使用生产者和消费者的API。对于复杂的数据变换，Kafka提供了Streams　API。Stream　API允许应用做一些复杂的处理，比如将流数据聚合或者join。

这一功能有助于解决以下这种应用程序面临的问题：处理无序数据，当消费端代码变更后重新处理输入，执行有状态计算等。

Streams API建立在Kafka的核心之上：它使用Producer和Consumer API作为输入，使用Kafka进行有状态的存储，并在流处理实例之间使用相同的消费组机制来实现容错。

### 1.8 批处理

Kafka将消息、存储和流处理结合起来，通过组合存储和低延迟订阅，流式应用程序可以以同样的方式处理过去和未来的数据。

同样，作为流数据管道，能够订阅实时事件使得Kafka具有非常低的延迟；同时Kafka还具有可靠存储数据的特性，可用来存储重要的支付数据，或者与离线系统进行交互，系统可间歇性加载数据，也可在停机维护后再次加载数据。流处理功能使得数据可以在到达时转换数据。


## 2.0 基本教程

如果是windows平台，使用bin\windows\而不是bin/，并将脚本扩展名改为.bat

### Step 1: 下载

[下载](https://www.apache.org/dyn/closer.cgi?path=/kafka/1.0.0/kafka_2.11-1.0.0.tgz)相关版本并解压缩。

> cd kafka  

### Step 2: 启动服务器

Kafka使用[ZooKeeper](https://zookeeper.apache.org/),如果你还没有ZooKeeper服务器。

> bin/zookeeper-server-start.sh config/zookeeper.properties  
> [2021-07-2 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)

开始启动Kafka服务器：

> bin/kafka-server-start.sh config/server.properties

### Step 3: 创建一个topic

创建一个名为"test"的topic，有一个分区和一个副本：

> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

现在运行list命令来查看这个topic:

> bin/kafka-topics.sh --list --zookeeper localhost:2181  

> test

### Step 4: 发送消息

Kafka自带一个命令行客户端，它从文件或标准输入中获取输入，将其作为message发送到Kafka集群。默认情况下，每行将作为单独的message发送。

运行producer,然后在控制台输入一些消息已发送到服务器。

> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

> This is a message

### Step 5: 启动一个consumer

Kafka有一个命令行consumer，将消息转储到标准输出。

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

> This is a message

### Step 6: 设置多个代理集群

首先，为每个代理创建一个配置文件（在windows上使用copy命令来代替）：

> cp config/server.properties config/server-1.properties  
> cp config/server.properties config/server-2.properties

### Step 7: 使用Kafka Connect来导入/导出数据

Kafka Connect是Kafka的一个工具，它可以将数据导入和导出到Kafka。它是一种可扩展工具，通过运行connectors（连接器）， 使用自定义逻辑来实现与外部系统的交互。 在本文中，我们将看到如何使用简单的connectors来运行Kafka Connect，这些connectors 将文件中的数据导入到Kafka topic中，并从中导出数据到一个文件。

首先，创建一个测试文件：

> echo -e "foo\nbar" > test.txt

在Windows系统使用：

> echo foo> test.txt  
> echo bar>> test.txt

接下来，我们将启动两个standalone（独立）运行的连接器，这意味着它们各自运行在一个单独的本地专用 进程上。 我们提供三个配置文件。首先是Kafka Connect的配置文件，包含常用的配置，如Kafka brokers连接方式和数据的序列化格式。 其余的配置文件均指定一个要创建的连接器。这些文件包括连接器的唯一名称，类的实例，以及其他连接器所需的配置。

> bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties

这些包含在Kafka中的示例配置文件使用您之前启动的默认本地群集配置，并创建两个连接器： 第一个是源连接器，用于从输入文件读取行，并将其输入到 Kafka topic。 第二个是接收器连接器，它从Kafka topic中读取消息，并在输出文件中生成一行。

在启动过程中，你会看到一些日志消息，包括一些连接器正在实例化的指示。 一旦Kafka Connect进程启动，源连接器就开始从 test.txt 读取行并且 将它们生产到主题 connect-test 中，同时接收器连接器也开始从主题 connect-test 中读取消息， 并将它们写入文件 test.sink.txt 中。我们可以通过检查输出文件的内容来验证数据是否已通过整个pipeline进行交付：

> more test.sink.txt

> foo

> bar

数据存储在Kafka topic connect-test 中，因此我们也可以运行一个console consumer（控制台消费者）来查看 topic 中的数据（或使用custom consumer（自定义消费者）代码进行处理）：

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning

> {"schema":{"type":"string","optional":false},"payload":"foo"}

> {"schema":{"type":"string","optional":false},"payload":"bar"}


连接器一直在处理数据，所以我们可以将数据添加到文件中，并看到它在pipeline 中移动：

> echo Another line>> test.txt

### Step 8: 使用Kafka Streams来处理数据

Kafka Streams是用于构建实时关键应用程序和微服务的客户端库，输入与输出数据存储在Kafka集群中。 Kafka Streams把客户端能够轻便地编写部署标准Java和Scala应用程序的优势与Kafka服务器端集群技术相结合，使这些应用程序具有高度伸缩性、弹性、容错性、分布式等特性。

## 3.0 使用案例

### 3.1 消息

Kafka很好的替代了传统的message broker（消息代理）。Message brokers可用于各种场合（如将数据生成器与数据处理解耦，缓冲未处理的消息等）。与大多数消息系统相比，Kafka拥有更好的吞吐量、内置分区、具有复制和容错功能，这使他成为一个非常理想的大型消息处理应用。

### 3.2 跟踪网站活动

Kafka的初始用例是将用户活动跟踪管道重建为一组实时发布-订阅源。这意味着网站活动（浏览网页、搜索或其他的用户操作）将被发布到中心topic，其中每个活动类型有一个topic。这些订阅源提供一系列用例，包括实时处理、实时监视、对加载到Hadoop或离线数据仓库系统的数据进行离线处理和报告等。

### 3.3 度量

Kafka通常用于监控数据。这涉及到从分布式应用程序中汇总数据，然后生成可操作的集中数据源。

### 3.4 日志聚合

日志聚合系统通常从服务器收集物理日志文件，并将其置于一个中心系统（可能是文件服务器或HDFS）进行处理。Kafka从这些日志文件中提取信息，并将其抽象为一个更加清晰的消息流。这样可以实现更低的延迟处理且易于支持多个数据源及分布式数据的消耗。

### 3.5 流处理

Kafka用户通过管道来处理数据，有多个阶段： 从Kafka topic中消费原始输入数据，然后聚合，修饰或通过其他方式转化为新的topic， 以供进一步消费或处理。Kafka Streams是一个轻量但功能强大的流处理库。

### 3.6 采集日志

[Event sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)是一种应用程序设计风格，按时间来记录状态的更改。 Kafka 可以存储非常多的日志数据，为基于 event sourcing 的应用程序提供强有力的支持。

### 3.7 提交日志

Kafka 可以从外部为分布式系统提供日志提交功能。 日志有助于记录节点和行为间的数据，采用重新同步机制可以从失败节点恢复数据。 Kafka的日志压缩 功能支持这一用法。

[有关Kafka提供的保证、API和功能的更多信息，请查看相关文档。](https://kafka.apache.org/documentation/#gettingStarted)