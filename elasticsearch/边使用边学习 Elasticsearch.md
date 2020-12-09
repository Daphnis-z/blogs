# 基本介绍

ES 核心技术栈：

- Elasticsearch

  存储和索引数据，使用 Java 编写并使用 Lucene 来建立索引并实现搜索功能。

  本质上是一个分布式存储，也是一个搜索引擎。

  补充：

  ​	Lucene ——迄今为止 性能最好、功能最全、最先进 的搜索引擎库

- Kibana

  管理、分析和展示 ES 中的数据

- Logstash

  采集数据，以及对数据进行预处理

- Beats

  作为数据采集代理部署到各个目标服务器上，可以采集 日志文件、网络数据包、系统和平台的各种度量 等，然后发往 logstash 进行后续处理

使用 ELKB 可以实现 日志、服务器性能、系统状态 等多种场景的 数据采集、处理、分析和展示，是一个**完整的生态链**。

ES 特点：

- 近实时的数据延迟（可以配置数据的刷新时间）
- 秒级的搜索速度
- 集群节点极易扩展
- 对搜索结果进行打分
- 无需预定义数据的 schema

ES 索引切分方案：

- 基于时间，根据数据量或业务需求 按天、月、年 都可，**最常见**
- 将数据唯一 ID 转换成一个数值，然后对一个数取余（10,100 ——根据需要来），最终将数据尽可能的均匀分布到各个索引
- 将数据按照批次 ID 分布到各个索引，适合导入离线数据的场景，一批数据有问题可以很方便的删除重来

# 名词解释

- Cluster

  集群，由一个或多个节点（node）组成，可以在 config/elasticsearch.yml 里修改集群的名称

- Node

  单个 es 实例，一般情况下一台服务器上运行一个 node 

  根据 node 作用可以分为以下几类：

  - master-eligible

    主节点，可以管理整个集群的设置及变化：创建，更新，删除 index；添加或删除 node；为 node 分配 shard

  - data

    数据节点

  - ingest

    数据接入节点

  - machine learning

    机器学习节点

  一个节点，可以同时具有多种角色。比如 主节点同时可以是数据节点

- Document

  文档，是 es 中索引或搜索的最小数据单元，可以简单的理解为一条数据，通常以 json 格式进行组织

- Type

  类型，是文档的逻辑容器，与文档的关系可以简单理解为关系型数据库中 表与行的关系

  类型默认是 _doc，在 es 6.0 以后一个索引只能有一个类型，而在未来的 es 8.0 后 类型这个概念将会被移除，即索引里面不再有类型

  类型被去除的原因如下：

  In an Elasticsearch index, fields that have the same name in different mapping types are backed by the same Lucene field internally. In other words, using the example above, the `user_name`field in the `user` type is stored in exactly the same field as the `user_name` field in the `tweet` type, and both `user_name` fields must have the same mapping (definition) in both types.

  即 不同类型中名字相同的两个字段在 lucene 中是同一个字段，这样将会导致在索引字段时会出现问题

- Index

  索引，是文档的集合，与文档的关系可以简单的理解为关系型数据库中 数据库与行的关系

  索引和数据库的区别在于：索引是一个逻辑命名空间，可以映射到一个或多个主分片，可以具有零个或多个副本分片

- Shard

  分片，为了使索引可以存储超过单个节点硬件限制的数据，可以将索引划分成多个分片，存储到不同的节点

  分片有两种类型：

  - primary shard

    主分片，每个文档都存储在一个主分片上。索引文档时，首先在主分片上编制索引，然后再同步到副本分片

    注意 创建索引后无法更改主分片的个数

  - replica shard

    副本分片，每个主分片可以有若干个副本分片。副本分片的作用主要有两个：高可用，当主分片不可用时，可以替代主分片工作；提高性能，可以替主分片处理 get 和 search 请求

- Replica

  副本，es 默认为每个索引创建一个主分片和一个副本分片

# 数据类型

keyword

所有字符都被当做一个字符串，在建立文档时，不需要进行 index。keyword 字段用于精确搜索，aggregation 和排序（sorting）。

# 健康状态

- 红色，集群中有主分片没有分配
- 黄色，已分配所有主分片，但是有副本分片没有分配
- 绿色，所有主分片和副本分片都已经分配完成

# 查询类型

## match_all

## match

## multi_match

## prefix

## term

## terms





# Kibana 高频查询语法

## 查询时过滤返回的字段

### 查询时指定 需要返回的字段

```
GET twitter/_search
{
  "_source": ["user", "city"],
  "query": {
    "match_all": {
    }
  }
}
```

### 查询时指定 需要返回的字段和不需要的字段

```
GET twitter/_search
{
  "_source": {
    "includes": [
      "user*",
      "location*"
    ],
    "excludes": [
      "*.lat"
    ]
  },
  "query": {
    "match_all": {}
  }
}
```

## 统计索引里的数据量

### 统计索引里数据总量

```
GET twitter/_count
```

### 统计符合条件的数据量

```
GET twitter/_count
{
  "query": {
    "match": {
      "city": "北京"
    }
  }
}
```

## 查询索引的映射

```
GET twitter/_mapping
```

## 查询指定字段含有特定前缀的数据

```
GET twitter/_search
{
  "query": {
    "prefix": {
      "user": {
        "value": "朝"
      }
    }
  }
}
```











# 参考资料

1. [Elastic 官方博客](https://blog.csdn.net/UbuntuTouch)
2. [Kibana 查询语法介绍](https://blog.csdn.net/UbuntuTouch/article/details/99546568)