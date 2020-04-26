---
title: Future
date: 2020-03-30 00:00:00
author: vanliuzh
top: false
cover: false
toc: true
mathjax: false
categories: Java
tags: [Java, Note]
reprintPolicy: cc_by
---


1. 先删缓存再改数据库

可能会有这样的问题

并发，1读，1写

- `写1`先删了`缓存`
- 这个时候`读1`发现没`缓存`，就读数据库准备写到`缓存`
- 然后`写1`要把数据写到`数据库`
- 接着`读1`把自己读到的数据写到`缓存`

这样缓存是`读1`放入的旧数据，数据库是`写1`放入的新数据，出现不一致

2. 先读数据库，再删除缓存

- 一开始没有缓存，读写并发
- 读操作先进来，发现没有缓存，去数据库中读数据，这个时候因为某种原因卡了，没有及时把数据放入缓存
- 写的操作进来了，修改了数据库，删除了缓存
- 读操作恢复，把老数据写进了缓存

这种情况下，读操作本来是要马上写缓存的，但是被写操作抢先了，写了新的数据进入，写操作的删除缓存没意义了，被后面读操作覆盖了

其实这种情况很极端了，但是会存在，因为并发，操作不是原子的

但是写一般是比较慢的，除非你读慢于写，才会发生

3. 延迟双删

延迟双删就是写中，先删除缓存，后修改数据库，最后延迟一定时间，再次删除缓存

这样写保证缓存不是旧缓存，是2情况的一种延时策略，缓存由读操作去建立，写操作保证不要写入旧缓存，否则读一直是旧缓存

4. 内存队列，消息队列

在3中，如果第二次删除失败了，那么就会有2的情况发生，上面3点都存在以下问题

- 修改数据库、删除缓存这两个操作耦合在了一起，没有很好的做到单一职责；
- 如果写操作比较频繁，可能会对Redis造成一定的压力；
- 如果删除缓存失败，该怎么办？

写操作只是修改数据库，然后把数据的Id放在内存队列里面，后台会有一个线程消费内存队列里面的数据，删除缓存，如果缓存删除失败，可以重试多次

生产考虑用消息队列，这样各个部门的数据都可以入队列，然后在容器或者整个服务里面启动一个服务，专门去消费队列，删除缓存

很好的解决了缓存不一致的问题，但是存在一定的延迟

说了这么多，有没有发现有点CAP的影子，保证了可用性，缺牺牲了一致性，但是保证了最终一致性

## 事件多播器

## 让你去实现一个注册中心，你会怎么做？

## 让你实现一个消息中间件，比如kafka，你会怎么做？

## dubbo调用的请求参数和返回参数无法使用泛型是因为什么

## dubbo 知识点

service 标签 暴露的服务
reference 标签 服务引用，标识从注册用心引用其它服务给自己用

推荐使用 Hessian 序列化

## zk

容器连接客户端

docker exec -it zookeeper /opt/zookeeper-3.4.13/bin/zkCli.sh -server 127.0.0.1:2181
docker exec -it zookeeper /opt/zookeeper-3.4.13/bin/zkServer.sh status

## zk选举

通常节点数要是偶数，因为集群过半投票后，才能选出新的leader，选不出就不提供服务

脑裂: 实际是不会有脑裂的，因为不满足过半条件，集群不再提供服务（过半机制防止脑裂）

假设有5台，过半就是5/2余数2，要大于2才行，也就是3，不能是等于2。也就是说集群必须存活着3台才能选举出新的leader，所以在5台组成的集群中，只能挂2台

在考虑选举的时候，是所有的，包括挂了的作为总数来计算，上面的例子就是5。假设总共有6台，分在2个机房，一个机房A 3台，另一个B也是3台，由于6/2等3，需要4台才能选出leader，所以AB机房都选不出，服务不可用。如果是AB分配是4，2那么A机房能选出leader，整个集群只有一个leader，这也是过半原则要大于的原因，如果是大于等于，在AB分配3，3的情况下，AB都满足等于3，都会选出leader，那就脑裂了

还有一个知识点就是部署最好是奇数，虽然奇数，偶数的集群都能选出leader，都不会脑裂，但是奇数能节约资源

5的情况下，运行挂2台(5/2 余 2，需要3台才能进行选举，所以容错是2台)，6的情况下，也是运行挂2台，那么我用5台就好了，节约资源

## Spring

源码流程

AnnotationConfigApplicationContext 通过注解配置创建context

在构造函数中，做了下面的事情

// 1.初始化bean定义读取器和扫描器
// 2.调用父类GenericApplicationContext无参构造函数，初始化一个BeanFactory：DefaultListableBeanFactory
// 3.注册Spring自带的bean，共5个 包括: 
//  ConfigurationClassPostProcessor
//  AutowiredAnnotationBeanPostProcessor  CommonAnnotationBeanPostProcessor
// 	EventListenerMethodProcessor  DefaultEventListenerFactory

## 分页

用starter风格的依赖，自动配置，除非你要显示的修改参数

然后直接创建分页对象PageHelper.startPage(pageNum, pageSize)，传递分页参数，然后获取查询对象，然后把查询对象赋值给pageInfo，返回分页数据(返回的数据中有很多字段，最好重新封装一下)

1. 数据结构，算法

看视频部分，一次过，总结一些小东西即可，B+树，红黑树要会，还有hash表，堆

以前用过的一致性hash，hash环

2. 基础

基础语法
反射，代理，class对象
新语法，lambda，流处理

2. 语言扩展相关

jvm,juc（并发容器，并发编程，AQS等）

3. 框架，构建工具

maven，Spring，Spring boot

4. 数据库

MySQL 原理，存储引擎，innodb，索引，锁，事务，事务隔离

读写分离，主从同步

框架结合: 读写分离配置，多数据源配置，事务声名怎么用，分布式事务怎么用

优化，扩展

innodb_flush_log_at_trx_commit MySql日志何时写入硬盘的参数
load data infile 从文件把数据写入表中
存储过程upset，适合去重，利用upset，有就更新，无就创建
ETL，是英文Extract-Transform-Load的缩写，用来描述将数据从来源端经过抽取（extract）、转换（transform）、加载（load）至目的端的过程。ETL一词较常用在数据仓库，但其对象并不限于数据仓库。

5. Redis

集群，数据结构，红锁，分布式事务

6. 中间件

kafka,rocketmq

7. 微服务,分布式，soa

k8s, spring cloud ，dubbo

8. 能力开放平台

9. 分布式锁


链接:https://pan.baidu.com/s/1pCViJ2lYrrl-k3-ic8ADZg  密码:mimx

