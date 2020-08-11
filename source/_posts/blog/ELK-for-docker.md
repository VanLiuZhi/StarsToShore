---
title: ELK实践与运用（部署）
date: 2020-08-11 00:00:00
author: vanliuzh
top: true
cover: false
toc: true
mathjax: false
tags: [Technology, Docker, Note]
categories: Technology技术
reprintPolicy: cc_by
---

日志处理作为系统基础设施的一环，有着举足轻重的作用，一般常用ELK作为日志处理平台。ELK是ElEasticsearch、Logstash、Kibana的简称，这三者是核心套件，但是随着功能的迭代和组件的更替，也出现了ELKF，其中的F指的是filebeat，也可以在ELK中加入Kafka组件，以满足复杂场景的需求，所以我们通常说的ELK泛指与之相关的各种组件

## 概述

为什么要做日志采集？

当我们的系统不是很复杂的时候，假设是单机架构的系统，以一个spring boot项目为例，我们可以通过logback做日志切割，把整个服务运行过程中的日志记录到文件中，方便后续排查问题。但是这样也有很多问题，

1. 比如日志在服务器上，而开发人员才需要看日志，一般都是运维做服务器管理的，所以有些小伙伴会开发日志下载功能，方便开发人员查看。不过日志都是实时产生的，通过下载文件查看日志的做法并不具备实时性

2. 当上到分布式系统的时候，一个业务的完成流程可能由多个服务参与，日志文件也是散落在各个服务器上的，需要把日志统一到一起才方便排查问题

ELK就提供了这样的解决方案，整体的思路是通过Logstash把日志采集后，统一发到ElEasticsearch，ElEasticsearch是一个高效的搜索引擎，然后我们通过官方提供的Kibana UI组件，去获取ElEasticsearch中的数据

下面我们来搭建一套ELK环境，全部采用Docker部署

## ElEasticsearch 安装

通过官方镜像来部署，ElEasticsearch可以很容易扩容，通过集群的能力提高系统对数据的处理能力，我们部署3台节点

### 资源规划

由于通过Docker部署的，jvm参数会在容器启动的时候固定(虽然可以通过非常规手段来调整，但是一般不建议这么做)，所以我们要提前规划好资源

建议每日日志在1亿左右规模的系统，由于数据量不大，用3个节点即可，每台配置4C 8G，硬盘100G

有一个非常重要的配置就是es jvm最大堆不要超过内存的一半（官方建议即使物理机内存达到128GB，也不要设置为64GB，最大设置到32GB）


