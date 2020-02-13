---
title: Spring Cloud 微服务实践
date: 2019-11-01 00:00:00
author: vanliuzh
top: true
cover: false
toc: true
mathjax: false
summary: Spring Cloud 微服务实践
categories: Java
tags: [Java, Note]
reprintPolicy: cc_by
---

## 版本简介

spring cloud的版本比较特殊，它不是常见的版本号形式，而是由英文单词+SR+数字的形式，英文单词由伦敦地铁明明。这种版本命名也不是什么新鲜事，比如英伟达就用物理学家名字命名自己的GPU架构

Spring Cloud版本发布记录可详见：https://github.com/spring-cloud/spring-cloud-release/releases ，从中我们可清晰看到Spring Cloud版本发布的时间及先后顺序

Spring Cloud版本演进计划：https://github.com/spring-cloud/spring-cloud-release/milestones ，从中我们可了解Spring Cloud的版本演进计划，例如计划什么时间点发布什么版本等

## SOA

面向服务的架构（SOA）Service-Oriented Architecture，它将应用程序的不同功能进行拆分，并通过服务之间定义的接口联系起来，一般来说接口定义应该是中立的，即独立于平台和编程语言的，这样使得不同语言的服务能有一种统一的方式进行交互

## 各种组件

Eureka: Spring Cloud用来提供服务发现和注册的组件
Ribbon: 提供服务负载均衡的组件
Feign: 封装服务直接的请求调用，不用指定point，这样服务ip端口变化时仍然能够访问
Hystrix: 提供服务熔断, Turbine做集群监控
Zuul: 提供服务网关
Config: 提供统一服务配置
Sleuth: 提供服务调用链，用于排查问题，和ZipKin配合使用，ZipKin提供可视化页面

Consul: 是一个服务网格（微服务间的 TCP/IP，负责服务之间的网络调用、限流、熔断和监控）解决方案，它是一个一个分布式的，高度可用的系统，而且开发使用都很简便。它提供了一个功能齐全的控制平面，主要特点是：服务发现、健康检查、键值存储、安全服务通信、多数据中心

## Spring Cloud Config

是一种统一配置管理的方案，简单来说要这样使用: 部署一个config service，这个服务有各个微服务需要用到的配置文件信息，然后可以通过restful或client与config service进行交互，这样各个微服务就能获取到配置参数了

对于这个config service它是可以动态修改的，可以高可用；另外还有一个概念就是配置文件的存储方式，有git，mysql，本地文件系统等形式

### git

项目结构规划

config-service 的client配置的连接service端的配置文件要写bootstrap中，原因如下:

1. bootstrap.yml文件中的内容不能放到application.yml中，否则config部分无法被加载
2. 因为config部分的配置先于application.yml被加载，而bootstrap.yml中的配置会先于application.yml加载
3. bootstarp配置文件是从云端加载配置文件。优先级高于application，项目启动时，会先去加载自带的配置文件(框架自己的)，然后加载bootstrap配置文件，将加载到的内容放入到application中

上面是什么意思呢？这是一个很重要的点，首先对于config-service来说，正常部署使用即可；而对于依赖于config-service配置的其它微服务来说，由于系统启动的时候就需要加载配置文件，绑定config-server的URL，然后在加载application配置。如果bootstrap文件找不到或者没有配置server的URL，系统会默认URL为http://127.0.0.1:8888(当然是你使用spring-cloud-starter-config依赖才是这个逻辑)

我们直接配置在application中就会导致读取不到配置文件，这个时候的报错大多数都是读取不到配置文件的错误，另外可能有人把配置文件在application也准备一份，这种情况不会报错(因为读取不到，使用了本地的，编译通过)，但是client是没有使用config-service的，这种情况也是要注意的

当我们在代码中使用配置文件的值绑定到属性的时候，service端有用service，没有用本地，本地没有报错


## 触发自我保护机制

部署起spring cloud相关组件后，在Eureka页面有时会报这个异常

`EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.`

Euerka Service和Euerka Client之间，默认30秒进行一次心跳，出现这个异常是因为client少于一定的阈值，server不会删除注册信息，这就有可能导致在调用微服务时，实际上服务并不存在。 这种保护状态实际上是考虑了client和server之间的心跳是因为网络问题，而非服务本身问题，不能简单的删除注册信息

