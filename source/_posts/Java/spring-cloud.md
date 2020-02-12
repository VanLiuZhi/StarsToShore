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


## 触发自我保护机制

部署起spring cloud相关组件后，在Eureka页面有时会报这个异常

`EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.`

Euerka Service和Euerka Client之间，默认30秒进行一次心跳，出现这个异常是因为client少于一定的阈值，server不会删除注册信息，这就有可能导致在调用微服务时，实际上服务并不存在。 这种保护状态实际上是考虑了client和server之间的心跳是因为网络问题，而非服务本身问题，不能简单的删除注册信息

