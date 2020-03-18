---
title: Kafka 学习笔记
date: 2019-03-19 00:00:00
author: vanliuzh
top: false
cover: false
toc: true
mathjax: true
categories: Middleware
tags: [Linux, Note]
reprintPolicy: cc_by
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

`数据结构`: 每个日志文件都是一个log entry序列，每个log entry包含一个4字节整型数值（值为N+5），1个字节的”magic value”，4个字节的CRC校验码，其后跟N个字节的消息体。每条消息都有一个当前Partition下唯一的64字节的offset，它指明了这条消息的起始位置。磁盘上存储的消息格式如下：
message length ： 4 bytes (value: 1+4+n)
“magic” value ： 1 byte
crc ： 4 bytes
payload ： n bytes

存储文件展示(一个topic下的文件)

![image](/images/Web/kafka/kafka-3.png)

`偏移量`：每条消息都有一个当前Partition下唯一的64字节的offset，它指明了这条消息的起始位置。从这个偏移量就可以知道消息在硬盘中的位置

因为每条消息都被append到该Partition中，属于顺序写磁盘，因此效率非常高（经验证，顺序写磁盘效率比随机写内存还要高，这是Kafka高吞吐率的一个很重要的保证）

`高性能`: 因为Kafka读取特定消息的时间复杂度为O(1)，即与文件大小无关，所以删除过期文件与提高Kafka性能无关。选择怎样的删除策略只与磁盘以及具体的需求有关。另外，Kafka会为每一个Consumer Group保留一些metadata信息----当前消费的消息的position，也即offset。这个offset由Consumer控制。正常情况下Consumer会在消费完一条消息后递增该offset。当然，Consumer也可将offset设成一个较小的值，重新消费一些消息。因为offet由Consumer控制，所以Kafka broker是无状态的，它不需要标记哪些消息被哪些消费过，也不需要通过broker去保证同一个Consumer Group只有一个Consumer能消费某一条消息，因此也就不需要锁机制，这也为Kafka的高吞吐率提供了有力保障

Producer发送消息到broker时，会根据Paritition机制选择将其存储到哪一个Partition。如果Partition机制设置合理，所有消息可以均匀分布到不同的Partition里，这样就实现了`负载均衡`。如果一个Topic对应一个文件，那这个文件所在的机器I/O将会成为这个Topic的性能瓶颈，而有了Partition后，不同的消息可以并行写入不同broker的不同Partition里，极大的提高了吞吐率

`消费组`：如果需要实现多播，只要每个Consumer有一个独立的Group就可以了。要实现单播，只要所有的Consumer在同一个Group里。一个Group中的多个consumer不能同时消费一个分区，严格来说kafka不支持广播，rocketMQ可以广播

为了保证高可用，`副本数会被分配到其它broker上`。就是当只有一个broker的时候，你不能创建分区的2个副本，kafk会把一个副本放到另一个分区上保证高可用。分区数量不受broker影响，主要是副本数

`副本与分区`: 关于这个副本与分区还要再强调一点，当我们设置一个分区的时候，不配置副本，此时副本数就是1，其实也就是分区数。 在broker中，会有一个partition，如果我们副本配置2，那么总共会看到2个partition，即partition_0，partition_1，其中一个是leader，一个是follower。所以副本设置1，或者不设置，分区就是一个，设置2，才会有2个分区。`broker中最多只能保存一个副本`。
**重点来了**: 当有2个broker，不能创建3个分区，系统报错；但是当我们有3个broker的时候，创建3个分区，然后停止一个broker，系统不会报错，被停止的broker上的分区进入到不可用状态

如下图通过命令可以看到副本和分区在broker中的分布情况

![image](/images/Web/kafka/kafka-1.png)

`Isr` isr(In-Sync Replicas)是分区的子集，表示副本的存活数，3个broker，我们踢出一个broker后，原来三个分区分布在3个broker上，现在只存活着2个

如图

![image](/images/Web/kafka/kafka-2.png)

`线程安全`: producer是线程安全的，consumer不是，consumer不能多线程操作，但是可以多线程用一个消费组，这是没有影响的


## 关于官网

http://kafka.apachecn.org/ 中文

http://kafka.apache.org/   英文

## 版本的改进

Kafka 0.8 之后引入了副本机制， Kafka 成为了一个分布式高可靠消息队列解决方案。
0.8.2.0 版本社区引入了新版本 Producer API ， 需要指定 Broker 地址，但是bug比较多
0.9.0.0 版本增加了基础的安全认证 / 权限功能，同时使用 Java 重写了新版本 Consumer API ，还 引入了 Kafka Connect 组件用于实现高性能的数据抽取，同样也是bug多
0.10.0.0 是里程碑式的大版本，因为该版本引入了 Kafka Streams。从这个版本起，Kafka 正式升 级成分布式流处理平台
0.11.0.0 版本，引入了两个重量级的功能变更:一个是提供幂等性 Producer API 以及事务 (Transaction) API ; 另一个是对 Kafka 消息格式做了重构。
1.0和2.0 主要是对Kafka Streams 的各种改进，在消息引擎方面并未引入太多的重大功能特性

最初的kafka是消息中间件，后面加入了流处理功能。1.1之后消息相关的代码就基本没动过了

## Docker 镜像

Kafka官方没有提供镜像，这里只能使用第三方的了，有些制作的还算不错的。

镜像分析，截取别人的经验：

1. wurstmeister/kafka  特点：star数最多，版本更新到 Kafka 1.0 ，zookeeper与kafka分开于不同镜像。

2. spotify/kafka  特点：star数较多，有很多文章或教程推荐，zookeeper与kafka置于同一镜像中；但kafka版本较老（还停留在0.10.1.0）。

3. confluent/kafka 背景：Confluent是书中提到的那位开发Kafka的Jay Kreps 从LinkedIn离职后创立的新公司，Confluent Platform 是一个流数据平台，围绕着Kafka打造了一系列产品。特点：大咖操刀，文档详尽，但是也和Confluent Platform进行了捆绑。

上述三个项目中，最简单的是spotify/kafka，但是版本较老。confluent/kafka 资料最为详尽，但是因为与Confluent Platform做了捆绑，所以略显麻烦。最终选定使用wurstmeister/kafka，star最多，版本一直保持更新，用起来应该比较放心。

## 命令

kafka 最新版本到 2.1.X 了

$KAFKA_HOME/bin/kafka-topics.sh --zookeeper zoo1:2181 --list // 查看所有主题
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list zoo1:9092 --topic test // 创建主题

$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test // 打开消息发送控制台
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 -topic test --from-beginning // 打开消息消费控制台

## 分区策略

Kafka 提供了消费者客户端参数partition.assignment.strategy来设置消费者与订阅主题之间的分区分 配策略。默认情况下，采用 Range Assignor 分配策略。 Kafka 还提供了另外两种分配策略: RoundRobinAssignor 和 StickyAssignor

`RangeAssignor` : RangeAssignor 分配策略的原理是按照消费者总数和分区总数进行整除运算来获得一个跨度，然后将分区按照跨度进行平均分配，以保证分区尽可能均匀地分配给所有的消费者

假设n=分区数/消费者数量，m=分区数%消费者数量，那么前m个消费者每个分配n+1个分区，后面的（消费者数量-m）个消费者每个分配n个分区。

为了更加通俗的讲解RangeAssignor策略，我们不妨再举一些示例。假设消费组内有2个消费者C0和C1，都订阅了主题t0和t1，并且每个主题都有4个分区，那么所订阅的所有分区可以标识为：t0p0、t0p1、t0p2、t0p3、t1p0、t1p1、t1p2、t1p3。最终的分配结果为：

```
消费者C0：t0p0、t0p1、t1p0、t1p1
消费者C1：t0p2、t0p3、t1p2、t1p3
```

这样分配的很均匀，那么此种分配策略能够一直保持这种良好的特性呢？我们再来看下另外一种情况。假设上面例子中2个主题都只有3个分区，那么所订阅的所有分区可以标识为：t0p0、t0p1、t0p2、t1p0、t1p1、t1p2。最终的分配结果为：

```
消费者C0：t0p0、t0p1、t1p0、t1p1
消费者C1：t0p2、t1p2
```

可以明显的看到这样的分配并不均匀，如果将类似的情形扩大，有可能会出现部分消费者过载的情况

其实这个例子举的也不是太好，可以理解为`从中间平均分开`，一刀划开，不够平分的情况，第一个消费组分到多余的

`RoundRobinAssignor` :RoundRobinAssignor 分配策略的原理是将消费组内所有消费者及消费者订阅的所有主题的分区按照字典序排序，然后通过轮询方式逐个将分区依次分配给每个消费者

如图所示
```
t1: 2   t1p0   t1p2
t2: 4   t2p0   t2p1    t2p2     t2p3

c0:     t1p0   t2p1
c1:     t1p2   t2p2
c2:     t2p0   t2p3
```

把所有分区排序后，一个一个的分配给消费者

其中一个consumer挂了，就把原来的分区也是排序后再分配给消费者

```
t1: 2   t1p0   t1p2
t2: 4   t2p0   t2p1    t2p2     t2p3

c0:     t1p0   t2p1
c1:     t1p2   t2p2     宕机
c2:     t2p0   t2p3

c0:     t1p0   t2p0    t2p2
c2:     t1p1   t2p1    t2p3
```

`StickyAssignor`: Kafka从 0.11.x 版本开始引入这种分配策略，它主要有两个目的: 分区的分配要尽可能均匀。分区的分配尽可能与上次分配的保持相同

采用RoundRobinAssignor会导致原来由自己消费的分区被分配给了其它消费者，而StickyAssignor就是解决这个问题，`分区的分配尽可能与上次分配的保持相同`

如图所示

```
t1: 2   t1p0   t1p2
t2: 4   t2p0   t2p1    t2p2     t2p3

c0:     t1p0   t2p1
c1:     t1p2   t2p2     宕机
c2:     t2p0   t2p3

c0:     t1p0   t2p1     t1p2
c2:     t2p0   t2p3     t2p2
```

`重复消费`: 分区策略，主要是要考虑重平衡的问题，假如其中一个消费组宕机，需要重新分配分区，这个消费组的分区还未提交偏移量，它的分区被分配给其它消费者来处理了，新的消费者从上个位点消费，这时候就重复消费了。重复消费的消息就是上个偏移量位点到原消费着未提交之前的消息

还有一种重复消费就是自动提交的情况下，消费者被关闭了，自动提交的时间间隔还没到，就还没来得及提交位移，下次启动消费者也会重复消费

重复消费可以通过监听器接口的实现来解决一些问题

```java
public interface ConsumerRebalanceListener{
    // 在rebalance开始之前和消费者停止读取消息之后被调用。可以通过这个回调方法 //来处理消费位移的提交， 以此来避免一些不必要的重复消费现象的发生。
    //参数 partitions表示rebalance前所分配到的分区。
    void onPartitionsRevoked(Collection<TopicPartition> partitions); // 在重新分配分区之后和消费者开始读取消费之前被调用。
    // 参数partitions表示再均衡后所分配到的分区。
    void onPartitionsAssigned(Collection<TopicPartition> partitions);
}
```

## 位移提交

这里聊的是消费者的，假如消息是1到10，现在消费到了5，提交，偏移量记消息6的偏移量，下次就从6的偏移量开始消费。这个6对应的偏移量，就是commit offset(上次提交的消费位移)

`消费者的位移也是保存在kafka的主题里面的`

每个consumer会定期将自己消费分区的offsets提交给kafka内部topic:consumer_offsets，提交过去 的时候，key是consumerGroupId+topic+分区号，value就是当前offset的值，kafka会定期清理topic里的消息，最后就保留最新的那条数据。因为consumer_offsets可能会接收高并发的请求，kafka默认给其分配50个分区(可以通过offsets.topic.num.partitions设置)，这样可以通过加机器的方式抗大并发


## 时间轮