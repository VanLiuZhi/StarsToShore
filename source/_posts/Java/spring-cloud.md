---
title: Spring Cloud 微服务实践
date: 2019-11-01 00:00:00
author: vanliuzh
top: true
cover: false
toc: true
mathjax: false
summary: Spring Cloud 微服务实践，是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等
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

`Eureka`: Spring Cloud用来提供服务发现和注册的组件
`Ribbon`: 提供服务负载均衡的组件
`Feign`: 封装服务直接的请求调用，不用指定point，这样服务ip端口变化时仍然能够访问
`Hystrix`: 提供服务熔断, Turbine做集群监控
`Zuul`: 提供服务网关
`Config`: 提供统一服务配置
`Sleuth`: 提供服务调用链，用于排查问题，和ZipKin配合使用，ZipKin提供可视化页面

`Consul`: 是一个服务网格（微服务间的 TCP/IP，负责服务之间的网络调用、限流、熔断和监控）解决方案，它是一个一个分布式的，高度可用的系统，而且开发使用都很简便。它提供了一个功能齐全的控制平面，主要特点是：服务发现、健康检查、键值存储、安全服务通信、多数据中心

## Eureka

如何使用: 启动一个eureka service，各个微服务通过配置client把自己注册到eureka service

eureka service主要配置，起多个服务实现高可用，注意它是CA架构。下面通过一个工程的不同profiles启动多个service实现高可用，如果是部署在一台主机上，hostname不能少

```yaml
eureka:
  client:
    # 由于是高可用，所以要同步数据
    # 是否要注册到其他Eureka Server实例
    register-with-eureka: true
    # 是否要从其他Eureka Server实例获取数据
    fetch-registry: true
    service-url:    
      defaultZone: http://localhost:8761/eureka/, http://localhost:8762/eureka/
---
spring:
  profiles: peer1
server:
  port: 8761
eureka:
  instance:
    hostname: peer1
---
spring:
  profiles: peer2
server:
  port: 8762
eureka:
  instance:
    hostname: peer2
```

client端加入服务

```yaml
eureka:
  client:
    service-url:
      # 指定eureka server通信地址，注意/eureka/小尾巴不能少，如果是有多个service，建议配置多个地址
      defaultZone: http://localhost:8761/eureka/
  instance:
    # 是否注册IP到eureka server，如不指定或设为false，那就会注册主机名到eureka server
    prefer-ip-address: true
```


## consule

目前，我对这个服务网格的理解还不是很深入，也看到有一些人分析这个东西并不是什么微服务的未来，总的来说比eureka更加底层的服务发现和注册

如何使用: 由于consule是go编写的，CP架构，部署完成后，Spring只需要关注clien如何加入即可完成集成

consule的功能比较丰富，也可以用它来代替config-service，依赖 `spring-cloud-starter-consul-config`

### 部署

consul agent -server -client=0.0.0.0 -bootstrap-expect=3 -data-dir=/Users/liuzhi/cloud/data/ -node=server1

consul agent -server -client=0.0.0.0 -bootstrap-expect=3 -data-dir=/Users/liuzhi/cloud/data/ -node=server2

consul agent -server -bind=0.0.0.2 -client=0.0.0.0 -bootstrap-expect=3 -data-dir=/Users/liuzhi/cloud/data/ -node=server3

## Spring Cloud Config

是一种统一配置管理的方案，简单来说要这样使用: 部署一个config service，这个服务有各个微服务需要用到的配置文件信息，然后可以通过restful或client与config service进行交互，这样各个微服务就能获取到配置参数了

对于这个config service它是可以动态修改的，可以高可用；另外还有一个概念就是配置文件的存储方式，有git，mysql，本地文件系统等形式

如何使用: 要启动一个cloud service服务，然后其它服务去这里读取配置文件，基本就是这么一个流程。

关于高可用: 
1. 存储仓库的高可用，如果是第三方git，本身高可用(除非你网络有问题)，当然实际肯定用自己搭建的，就需要准备高可用，存储也是，文件系统高可用

2. 服务高可用，可以使用注册中心，把多个service注册到服务中心，clien也是从注册中心读取配置。如果不用注册中心，就需要负载均衡器做高可用。当然soa架构，肯定是有注册中心的 

关于动态刷新: 这里我有个疑问，假如是通过`@Value`读取的配置文件，那么刷新后用新的值，确实没问题，但是像数据库配置这种，刷新后应该是要重启才行吧(简单属性和比较复杂的配置属性)，未完待续


### git

项目结构规划

config-service 的client配置的连接service端的配置文件要写bootstrap中，原因如下:

1. bootstrap.yml文件中的内容不能放到application.yml中，否则config部分无法被加载
2. 因为config部分的配置先于application.yml被加载，而bootstrap.yml中的配置会先于application.yml加载
3. bootstarp配置文件是从云端加载配置文件。优先级高于application，项目启动时，会先去加载自带的配置文件(框架自己的)，然后加载bootstrap配置文件，将加载到的内容放入到application中

上面是什么意思呢？这是一个很重要的点，首先对于config-service来说，正常部署使用即可；而对于依赖于config-service配置的其它微服务来说，由于系统启动的时候就需要加载配置文件，绑定config-server的URL，然后在加载application配置。如果bootstrap文件找不到或者没有配置server的URL，系统会默认URL为http://127.0.0.1:8888(当然是你使用spring-cloud-starter-config依赖才是这个逻辑)

可以看到引入依赖后，不配置的默认输出

```log
2020-02-20 17:13:40.418  INFO 36775 --- [  restartedMain] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://localhost:8888
2020-02-20 17:13:40.554  INFO 36775 --- [  restartedMain] c.c.c.ConfigServicePropertySourceLocator : Connect Timeout Exception on Url - http://localhost:8888. Will be trying the next url if available
2020-02-20 17:13:40.554  WARN 36775 --- [  restartedMain] c.c.c.ConfigServicePropertySourceLocator : Could not locate PropertySource: I/O error on GET request for "http://localhost:8888/application/default": Connection refused (Connection refused); nested exception is java.net.ConnectException: Connection refused (Connection refused)
```

我们直接配置在application中就会导致读取不到配置文件，这个时候的报错大多数都是读取不到配置文件的错误，另外可能有人把配置文件在application也准备一份，这种情况不会报错(因为读取不到，使用了本地的，编译通过)，但是client是没有使用config-service的，这种情况也是要注意的

当我们在代码中使用配置文件的值绑定到属性的时候，service端有用service，没有用本地，本地没有报错


## 触发自我保护机制

部署起spring cloud相关组件后，在Eureka页面有时会报这个异常

`EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.`

Euerka Service和Euerka Client之间，默认30秒进行一次心跳，出现这个异常是因为client少于一定的阈值，server不会删除注册信息，这就有可能导致在调用微服务时，实际上服务并不存在。 这种保护状态实际上是考虑了client和server之间的心跳是因为网络问题，而非服务本身问题，不能简单的删除注册信息

