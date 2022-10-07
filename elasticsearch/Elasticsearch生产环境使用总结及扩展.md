# 1.简介

下面的介绍摘自 Elasticsearch官网：

> Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。 作为 Elastic Stack 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。

这里根据笔者的经验，对整个 Elastic Stack进行简要介绍：

- **Elasticsearch**

  存储和索引数据，使用 Java 编写并使用 Lucene 来建立索引并实现搜索功能。

  本质上是一个分布式存储，也是一个搜索引擎。

  补充：

  ​	Lucene ——迄今为止 性能最好、功能最全、最先进 的搜索引擎库

- **Kibana**

  管理、分析和展示 ES 中的数据，比较常用的功能有 Discover, Visualize, Console和 Monitoring等。

- **Logstash**

  采集数据，以及对数据进行预处理。

- **Beats**

  作为数据采集代理部署到各个目标服务器上，可以采集 日志文件、网络数据包、系统和平台的各种度量 等，然后发往 logstash 进行后续处理。

针对本文的主角 es，再介绍下其主要特性和常见的索引切分方案：

- **主要特性**
  - 近实时的数据延迟（可以配置数据的刷新时间）
  - 秒级的搜索速度
  - 集群节点极易扩展
  - 对搜索结果进行打分
  - 无需预定义数据的 schema
- **索引切分**
  - 基于时间，根据数据量或业务需求 按天、月、年 都可 ——**最常见**
  - 将数据唯一 ID 转换成一个数值，然后对一个数取余（10,100 ——根据需要来），最终将数据尽可能的均匀分布到各个索引
  - 将数据按照批次 ID 分布到各个索引，适合导入离线数据的场景，一批数据有问题可以很方便的删除重来

# 2.名词解释

- **Cluster**

  集群，由一个或多个节点（node）组成，可以在 config/elasticsearch.yml 里修改集群的名称

- **Node**

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

- **Index**

  索引，是文档的集合，与文档的关系可以简单的理解为关系型数据库中 数据库与行的关系

  索引和数据库的区别在于：索引是一个逻辑命名空间，可以映射到一个或多个主分片，可以具有零个或多个副本分片

- **Inverted index**

  倒排索引，是 es和任何其他支持全文搜索的系统的核心数据结构。即 构建一个 词:文档ID 的字典，给定一个词可以快速知道这个词在哪些文档里出现过。

- **Document**

  文档，是 es中索引或搜索的最小数据单元，可以简单的理解为一条数据，通常以 json格式进行组织。

- **Type**

  类型，是文档的逻辑容器，与文档的关系可以简单理解为关系型数据库中 表与行的关系

  类型默认是 _doc，在 es 6.0以后一个索引只能有一个类型，而在未来的 es 8.0 后类型这个概念将会被移除，即索引里面不再有类型。

  类型被去除的原因如下：

  > In an Elasticsearch index, fields that have the same name in different mapping types are backed by the same Lucene field internally. In other words, using the example above, the `user_name`field in the `user` type is stored in exactly the same field as the `user_name` field in the `tweet` type, and both `user_name` fields must have the same mapping (definition) in both types.
  >

  即 不同类型中名字相同的两个字段在 lucene 中是同一个字段，这样将会导致在索引字段时会出现问题

- **Mapping**

  映射，存储了文档里各个字段及其的类型

- **Shard**

  分片，为了使索引可以存储超过单个节点硬件限制的数据，可以将索引划分成多个分片，存储到不同的节点

  分片有两种类型：

  - primary shard

    主分片，每个文档都存储在一个主分片上。索引文档时，首先在主分片上编制索引，然后再同步到副本分片。

    注意 创建索引后无法更改主分片的个数。

  - replica shard

    副本分片，每个主分片可以有若干个副本分片。副本分片的作用主要有两个：高可用，当主分片不可用时，可以替代主分片工作；提高性能，可以替主分片处理 get 和 search 请求。

- **Replica**

  副本，es 默认为每个索引创建一个主分片和一个副本分片。

- **Source**

  es 存储的原始数据就放在 source 里面。

- **Doc_values**

  在文档索引时构建的磁盘数据结构，存储与 _source相同的值，但以面向列（column）的方式存储，这对于排序和聚合而言更为有效。几乎所有字段类型都支持 Doc值，但对字符串字段除外 （text 及annotated_text）。Doc values 告诉你对于给定的文档 ID，字段的值是什么。

# 3.数据类型

这里主要介绍一些生产环境经常用到的数据类型。

- **keyword**

  > which is used for structured content such as IDs, email addresses, hostnames, status codes, zip codes, or tags.
  >

  适合存储短文本，所有字符都被当做一个字符串，在建立文档时，不需要进行 index。keyword 字段用于精确搜索，aggregation 和排序（sorting）。

- **text**

  > A field to index full-text values, such as the body of an email or the description of a product. These fields are `analyzed`, that is they are passed through an [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/analysis.html) to convert the string into a list of individual terms before being indexed. 
  >

  适合存储长文本，文本在存储之前，es 会对其进行分析，比如 切词。

  **如何关闭分析（禁止建立索引）？**

  在索引模板里面设置 index=false，如下：

  ```
  "content" : {
  "type" : "text",
  "index": false
  }
  ```

  不需要直接搜索的字段可以关闭索引，好处如下：

  - 增加数据插入时的速度，且降低插入时的资源消耗（不会进行 切词等分析操作）

  - 减少磁盘的占用

  关闭索引后并不会影响对该字段进行 aggregation 及对 source 的访问。

- **integer、double、long、boolean** ——简单类型

- **ip** ——ipv4 和 ipv6 都支持

- **date** ——日期类型

- **binary** ——以 Base64 编码的字符串

- **object** ——json 对象

- **geo_point**

  纬度和经度点，eg. (36.25,118.36)

补充：es 中并没有单独设置 数组类型，因为任何类型的字段都可以拥有 若干个值。

# 4.健康状态

生产环境通常会使用 kibana来查看 es集群的状态，而集群的健康状态会由不同的颜色来表示：

- **红色** ——**高度重视**，一定要解决，会影响部分索引的查询

  集群中有主分片没有分配。

- **黄色** ——尽量解决，可能会影响部分索引的查询效率

  已分配所有主分片，但是有副本分片没有分配。

- **绿色** ——all is well，集群十分健康

  所有主分片和副本分片都已经分配完成。

# 5.查询类型

## 5.1 match

1. 查询索引里所有的数据

   ```shell
   GET twitter/_search
   {
     "query": {
       "match_all": {
       }
     }
   }
   ```

2. 查询和指定字段中含有某个关键词的数据，结果按关联度由高到低排序

   ```shell
   GET twitter/_search
   {
     "query": {
       "match": {
         "city": "北京"
       }
     }
   }
   ```

3. 匹配短语，如下，hello 需要在 world 前面才行

   ```shell
   GET twitter/_search
   {
     "query": {
       "match_phrase": {
         "message": "hello world"
       }
     },
     "highlight": {
       "fields": {
         "message": {}
       }
     }
   }
   ```

4. 在多个字段中搜索含有某个关键词的数据

   ```shell
   GET twitter/_search
   {
     "query": {
       "multi_match": {
         "query": "朝阳",
         "fields": [
           "user",
           "address",
           "message"
         ],
         "type": "best_fields"
       }
     }
   }
   ```

## 5.2 prefix

查询指定的字段中含有特殊前缀的数据：

```shell
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

## 5.3 wildcard

通配符查询：

```shell
GET twitter/_search
{
  "query": {
    "wildcard": {
      "city.keyword": {
        "value": "*海"
      }
    }
  }
}
```

## 5.4 term

1. 查询和指定字段 精确匹配的数据

   ```shell
   GET twitter/_search
   {
     "query": {
       "term": {
         "user.keyword": {
           "value": "朝阳区-老贾"
         }
       }
     }
   }
   ```

2. 查询和指定字段 精确匹配的数据，可以指定多个关键词，不同关键词之间是 或 的关系

   ```shell
   GET twitter/_search
   {
     "query": {
       "terms": {
         "user.keyword": [
           "双榆树-张三",
           "东城区-老刘"
         ]
       }
     }
   }
   ```

## 5.5 range

范围查询，如下：

```shell
GET twitter/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 30,
        "lte": 40
      }
    }
  }
}
```

## 5.6 bool

可以把多个简单查询组合到一起，如下：

```shell
GET twitter/_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

**must**: 必须满足该条件

**must_not**: 必须不符合该条件

**filter**: 直接对数据进行过滤，不进行打分

**should**: 表达 或的意思，如果命中了会使得数据的相关性更高

## 5.7 geo_distance

位置查询，这是 es最擅长的：

```shell
GET twitter/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "北京"
          }
        }
      ]
    }
  },
  "post_filter": {
    "geo_distance": {
      "distance": "3km",
      "location": {
        "lat": 39.920086,
        "lon": 116.454182
      }
    }
  }
}
```

这里查找在地址栏里有“北京”，并且在以位置(116.454182, 39.920086)为中心的3公里以内的所有文档。

## 5.8 exists

查询指定字段不为空的数据：

```shell
GET twitter/_search
{
  "query": {
    "exists": {
      "field": "city"
    }
  }
}
```

## 5.9 sql

es 是支持使用 sql 进行查询的，如下：

```shell
GET /_sql?
{
  "query": """
    SELECT * FROM twitter 
    WHERE age = 30
  """
}
```

# 6.高频查询

这里对一些生产上经常使用的查询语法进行一些介绍。

## 6.1 查询时过滤返回的字段

1. 查询时指定 需要返回的字段

   ```shell
   GET twitter/_search
   {
     "_source": ["user", "city"],
     "query": {
       "match_all": {
       }
     }
   }
   ```

2. 查询时指定 需要返回的字段和不需要的字段

   ```shell
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

## 6.2 统计索引里的数据量

1. 统计索引里数据总量

   ```
   GET twitter/_count
   ```

2. 统计符合条件的数据量

   ```shell
   GET twitter/_count
   {
     "query": {
       "match": {
         "city": "北京"
       }
     }
   }
   ```

## 6.3 查询索引的映射

```
GET twitter/_mapping
```

## 6.4 查询指定字段含有特定前缀的数据

```shell
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

# 7.数据聚合

1. **Bucketing**

   构建存储桶的一系列聚合，其中每个存储桶与密钥和文档标准相关联。执行聚合时，将在上下文中的每个文档上评估所有存储桶条件，并且当条件匹配时，文档被视为“落入”相关存储桶。在聚合过程结束时，我们最终会得到一个桶列表 - 每个桶都有一组“属于”它的文档。

2. **Metric**

   聚合可跟踪和计算一组文档的指标。

3. **Martrix**

   一系列聚合，它们在多个字段上运行，并根据从请求的文档字段中提取的值生成矩阵结果。与度量标准和存储区聚合不同，此聚合系列尚不支持脚本。

4. **Pipeline**

   聚合其他聚合的输出及其关联度量的聚合

# 8.总结

本文从 基本概念、数据类型、查询类型、数据聚合、集群维护等维度对 Elasticsearch进行了简要介绍，更多详细内容，笔者建议直接前往 [Elasticsearch官网]([Elasticsearch：官方分布式搜索和分析引擎 | Elastic](https://www.elastic.co/cn/elasticsearch/)) 进行了解。