---
title: Kubernetes 笔记
date: 2019-04-05 00:00:00
tags: [linux, docker, note]
categories: web开发
---

Kubernetes 笔记

<!-- more -->

## 基础

协议规范:

CNI( Container Network Interface) 
CSI（Container Storage Interface） 
CRI（Container Runtime Interface）

etcd: 是一个高可用的分布式键值(key-value)数据库。etcd内部采用raft协议作为一致性算法，有点像zk，k8s集群使用etcd作为它的数据后端，etcd是一种无状态的分布式数据存储集群。数据以key-value的形式存储在其中

swap: 在k8s中最好禁用swap，以免发生集群无法调度问题

flannel: 用来通信的网络插件。要符号CNI规范

## k8s 用的的镜像示例

master节点

| Command                                                         | Description 
| --------------------------------------------------------------- | :---------------------------------: 
| REPOSITORY                                                      | TAGIMAGEIDCREATEDSIZE 
| registry.aliyuncs.com/google_containers/kube-proxy              | v1.14.120a2d703516510monthsago82.1MB 
| registry.aliyuncs.com/google_containers/kube-apiserver          | v1.14.1cfaa4ad74c3710monthsago210MB 
| registry.aliyuncs.com/google_containers/kube-controller-manager | v1.14.1efb3887b411d10monthsago158MB 
| registry.aliyuncs.com/google_containers/kube-scheduler          | v1.14.18931473d5bdb10monthsago81.6MB 
| quay-mirror.qiniu.com/coreos/flannel                            | v0.11.0-amd64ff281650a72112monthsago52.6MB 
| quay.io/coreos/flannel                                          | v0.11.0-amd64ff281650a72112monthsago52.6MB 
| registry.aliyuncs.com/google_containers/coredns                 | 1.3.1eb516548c18012monthsago40.3MB 
| registry.aliyuncs.com/google_containers/etcd                    | 3.3.102c4adeb21b4f14monthsago258MB 
| registry.aliyuncs.com/google_containers/pause                   | 3.1da86e6ba6ca12yearsago742kB 


node节点

| Command                                            | Description 
| -------------------------------------------------- | :------------------------------------------------: 
| REPOSITORY                                         | TAGIMAGEIDCREATEDSIZE 
| registry.aliyuncs.com/google_containers/kube-proxy | v1.14.120a2d703516510monthsago82.1MB 
| quay-mirror.qiniu.com/coreos/flannel               | v0.11.0-amd64ff281650a72112monthsago52.6MB 
| quay.io/coreos/flannel                             | v0.11.0-amd64ff281650a72112monthsago52.6MB 
| registry.aliyuncs.com/google_containers/pause      | 3.1da86e6ba6ca12yearsago742kB 


## 常用命令

kubectl explain node

kubectl get pods -n kube-system -o wide
kubectl get nodes -o wide  

`kubectl get node` 可以使用 -o 配合 wide json yaml

配合jq做内容提取

kubectl get nodes -o json | jq ".items[] | {name: .metadata.name} + .status.nodeInfo"

## 与k8s交互

在 K8S 中进行部署或者说与 K8S 交互的方式主要有三种：

命令式
命令式对象配置
声明式对象配置

## 镜像拉取策略

默认值是IfNotPresent

Always
总是拉取：
首先获取仓库镜像信息，如果仓库中的镜像与本地不同，那么仓库中的镜像会被拉取并覆盖本地。
如果仓库中的镜像与本地一致，那么不会拉取镜像。
如果仓库不可用，那么pod运行失败。

IfNotPresent
优先使用本地：
如果本地存在镜像，则使用本地的，
不管仓库是否可用。
不管仓库中镜像与本地是否一致。

Never
只使用本地镜像，如果本地不存在，则pod运行失败

## 固定node发布

deploy上用选择器，就可以固定在一个node上