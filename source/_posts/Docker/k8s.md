---
title: Kubernetes(k8s) 安装过程
date: 2019-04-05 00:00:00
tags: [linux, docker, note]
categories: web开发
---

Kubernetes(k8s) 安装过程

<!-- more -->

## 基本名称解释

etcd: 是一个高可用的分布式键值(key-value)数据库。etcd内部采用raft协议作为一致性算法，有点像zk，k8s集群使用etcd作为它的数据后端，etcd是一种无状态的分布式数据存储集群。数据以key-value的形式存储在其中