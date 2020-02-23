---
title: Java review
date: 2019-04-05 00:00:00
tags: [java, note]
categories: Java
---

review

<!-- more -->

## 表结构

innodb 表，数据和索引是一个文件
myisim 表，数据，索引，三个分开

为什么直接跳到8 滑稽

MySQL 5.5 -> MySQL 5
MySQL 5.6 -> MySQL 6
MySQL 5.7 -> MySQL 7
MySQL 8.0 -> MySQL 8

## 疑问

悲观锁用for_update
乐观锁 通过判断version，不是一般事务隔离是可重复读吗？那么多个线程读到的版本应该是一样的吧

把select语句放在事务中,查询的就是master主库了 感觉这句话说的不对，应该和框架有关

锁

共享锁
排它锁

意向共享锁
意向排它锁

间隙锁