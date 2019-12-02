---
title: elasticsearch 全文搜索解引擎在 web 开发中的运用
date: 2019-01-11 12:26:46
tags: [python, web]
categories: web开发
toc: true
---

搜索服务是Web开发中非常常见的服务，一般比较基本的会直接访问到数据库，但是如果要查询多个数据库才能得到搜索结果，这无疑加大了数据库的负担，
而且频繁的访问数据库容易成为应用的瓶颈。可以使用elasticsearch来提供搜索服务，降低数据库的压力。

<!-- more -->

## 概述

elasticsearch是基于Java的，所以在安装的时候可能需要Java环境，使用Docker来运行服务会比较简单。Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）。下面我们以单服务的形式在Docker中来使用它，为Web应用提供搜索服务支持。另外elasticsearch的操作其实是向服务发生请求。

## base

通俗的将Elastic就是一个数据库，它以索引Index为顶层单位，一个Index可以理解为一个数据库，其中里面的单条记录称为Document，同一个 Index 里面的 Document，不要求有相同的结构（scheme），但是最好保持相同，这样有利于提高搜索效率。

关于Tyoe：

1. Document 可以分组，比如weather这个 Index 里面，可以按城市分组（北京和上海），也可以按气候分组（晴天和雨天）。这种分组就叫做 Type，它是虚拟的逻辑分组，用来过滤 Document。

2. 不同的 Type 应该有相似的结构（schema），举例来说，id字段不能在这个组是字符串，在另一个组是数值。这是与关系型数据库的表的一个区别。性质完全不同的数据（比如products和logs）应该存成两个 Index，而不是一个 Index 里面的两个 Type（虽然可以做到）。

## 在Python中使用elasticsearch

首先，安装elasticsearch，这里我使用Docker来运行，默认端口为9200。然后安装elasticsearch-dsl，不要使用elasticsearch，因为elasticsearch是底层服务，我们需要使用像elasticsearch-dsl这样的高级封装库，它能更方便的编写和操作查询。

```python

from elasticsearch_dsl import Document, Integer, Text, Boolean, Q, Keyword, SF, Date
from elasticsearch_dsl.connections import connections

connections.create_connection(hosts=ES_HOSTS) # 连接服务

class Item(Document):
    id = Integer()
    title = Text()
    kind = Integer()
    content = Text()
    can_show = Boolean()
    created_at = Date()
    tags = Text(fields={'raw': Keyword()})

    class Index:
        name = 'test' # 指定Index的名称

    @classmethod
    def add(cls, item):
        obj = cls(**kwargs)
        obj.save()
        return obj
```

## bulk 进行批量操作

bulk相关的API可以在单个请求中一次执行多个操作(index,udpate,create,delete)，这比执行多次操作性能要好。相关API在使elasticsearch.helpers中，helpers是bulk的帮助程序，是对bulk的封装。

有三种方式bulk（），streaming_bulk（），parallel_bulk（）

```py
items = []
search = Items.search()
objects = ({
            '_op_type': 'update',
            '_id': f'{doc.id}_{doc.kind}',
            '_index': 'test',
            '_type': 'doc',
            '_source': doc.to_dict()
        } for doc in items)
client = connections.get_connection(search._using)
rs = list(parallel_bulk(client, objects,
                        chunk_size=500))
```

举例了parallel_bulk的用法，这里需要迭代才会执行，使用list

## 新内容

es 7.0 对应  head 插件 5.x  这是我上次的坑 如果不用5.x 的head 会连不上es

