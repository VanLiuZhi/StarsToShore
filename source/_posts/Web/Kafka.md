---
title: Kafka 学习笔记
date: 2019-03-19 00:00:00
tags: [linux, note]
categories: web开发
---

消息队列Kafka

<!-- more -->

## 基础概念和术语

**Partition** 分区，Partition是物理上的概念，每个Topic包含一个或多个Partition

**Consumer** 消费者 向Kafka broker读取消息的客户端
Consumer Group
每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）

**Topic** 每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）

**Broker** Kafka集群包含一个或多个服务器，这种服务器被称为broker

**Producer** 负责发布消息到Kafka broker

无法保证消息在一个主题内的顺序，但是可以保证消息在一个分区内的顺序，每个Partition在物理上对应一个文件夹，该文件夹下存储这个Partition的所有消息和索引文件


每个日志文件都是一个log entry序列，每个log entry包含一个4字节整型数值（值为N+5），1个字节的”magic value”，4个字节的CRC校验码，其后跟N个字节的消息体。每条消息都有一个当前Partition下唯一的64字节的offset，它指明了这条消息的起始位置。磁盘上存储的消息格式如下：
message length ： 4 bytes (value: 1+4+n)
“magic” value ： 1 byte
crc ： 4 bytes
payload ： n bytes

偏移量：每条消息都有一个当前Partition下唯一的64字节的offset，它指明了这条消息的起始位置。从这个偏移量就可以知道消息在硬盘中的位置

因为每条消息都被append到该Partition中，属于顺序写磁盘，因此效率非常高（经验证，顺序写磁盘效率比随机写内存还要高，这是Kafka高吞吐率的一个很重要的保证）

因为Kafka读取特定消息的时间复杂度为O(1)，即与文件大小无关，所以删除过期文件与提高Kafka性能无关。选择怎样的删除策略只与磁盘以及具体的需求有关。另外，Kafka会为每一个Consumer Group保留一些metadata信息——当前消费的消息的position，也即offset。这个offset由Consumer控制。正常情况下Consumer会在消费完一条消息后递增该offset。当然，Consumer也可将offset设成一个较小的值，重新消费一些消息。因为offet由Consumer控制，所以Kafka broker是无状态的，它不需要标记哪些消息被哪些消费过，也不需要通过broker去保证同一个Consumer Group只有一个Consumer能消费某一条消息，因此也就不需要锁机制，这也为Kafka的高吞吐率提供了有力保障。

Producer发送消息到broker时，会根据Paritition机制选择将其存储到哪一个Partition。如果Partition机制设置合理，所有消息可以均匀分布到不同的Partition里，这样就实现了负载均衡。如果一个Topic对应一个文件，那这个文件所在的机器I/O将会成为这个Topic的性能瓶颈，而有了Partition后，不同的消息可以并行写入不同broker的不同Partition里，极大的提高了吞吐率。

消费组：如果需要实现广播，只要每个Consumer有一个独立的Group就可以了。要实现单播只要所有的Consumer在同一个Group里。

## Docker 镜像

Kafka官方没有提供镜像，这里只能使用第三方的了，有些制作的还算不错的。

镜像分析，截取别人的经验：

1. wurstmeister/kafka  特点：star数最多，版本更新到 Kafka 1.0 ，zookeeper与kafka分开于不同镜像。

2. spotify/kafka  特点：star数较多，有很多文章或教程推荐，zookeeper与kafka置于同一镜像中；但kafka版本较老（还停留在0.10.1.0）。

3. confluent/kafka 背景：Confluent是书中提到的那位开发Kafka的Jay Kreps 从LinkedIn离职后创立的新公司，Confluent Platform 是一个流数据平台，围绕着Kafka打造了一系列产品。特点：大咖操刀，文档详尽，但是也和Confluent Platform进行了捆绑。

上述三个项目中，最简单的是spotify/kafka，但是版本较老。confluent/kafka 资料最为详尽，但是因为与Confluent Platform做了捆绑，所以略显麻烦。最终选定使用wurstmeister/kafka，star最多，版本一直保持更新，用起来应该比较放心。

## 命令

Harbor docker 的镜像管理仓库 企业级Registry服务器

kafka 最新版本到 2.1.X 了


$KAFKA_HOME/bin/kafka-topics.sh --zookeeper zoo1:2181 --list // 查看所有主题
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list zoo1:9092 --topic test // 创建主题

$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test // 打开消息发送控制台
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 -topic test --from-beginning // 打开消息消费控制台