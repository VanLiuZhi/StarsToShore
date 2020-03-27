---
title: Java Dubbo
date: 2019-02-05 00:00:00
author: vanliuzh
top: true
cover: false
toc: false
mathjax: true
categories: Java
tags: [Java, Note, Microservice]
reprintPolicy: cc_by
---

## Dubbo

## 关于协议的性能

各种协议的性能主要是2点来决定的

1. 码流大小
2. 编码解码效率

你像HTTP由于整个报文要包含请求头等信息，它的码流也会相应的增加