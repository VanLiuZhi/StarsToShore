---
title: Java review
date: 2019-04-05 00:00:00
tags: [java, note]
categories: Java
---

review

<!-- more -->

# problem

泊松分布的参数λ是单位时间(或单位面积)内随机事件的平均发生次数。 泊松分布适合于描述单位时间内随机事件发生的次数。

## 一致性哈希

先了解一个概念，取模和取余，如果除数都是正整数，那么取余和取模都是一样的，求余数
当 x 和 y 的正负号一样的时候，两个函数结果是等同的；当 x 和 y 的符号不同时，rem函数结果的符号和 x 的一样，而 mod 和 y 一样

在分布式中，或者用分库分表来说明，user表存在在3台机器上，用`用户id的hashcode % 3`来决定该用户的数据存储在哪台机器上。当机器增加到4台的时候，此时的模3就不对了，造成数据混乱。而一致性哈希算法就是解决这个问题的，一致性哈希保证当分布式环境的节点增加的时候，原来请求分配到的节点还是原来的节点


## Spring boot

配置: server.context-path 就是在请求上加个默认路径

@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
排除此类的autoconfig，比如我们引入了mybatis的依赖，按照Spring的规范，就要装配对应的bean，没有配置数据库会报错。这个时候就能用这个注解来取消某些类的自动装配

@EnableDiscoveryClient与@EnableEurekaClient
都是服务client用的，如果用eureka之外的，用EnableDiscoveryClient，主要是为了组件通用化，可以通过注解参数取消服务注册，新版本可以去掉注解，具体看使用哪个版本

spring-cloud-config-client
spring-cloud-starter-config