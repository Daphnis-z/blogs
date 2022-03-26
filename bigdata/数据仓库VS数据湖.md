# 1.前言

本文将新兴的**数据湖**技术和**数据仓库**技术进行了对比，然后简要介绍三种常见的数据湖实施方案。

# 2.数据仓库痛点

- **没有存储非结构化的数据**

  这里并不是说数仓**不能存储**非结构化的数据，而是数仓的分层模型决定了数据会被规整计算为结构化的数据，然后在处理完成的数据上进行建模、分析等。

  一般的数仓分层模型：ODS-> DWD-> DWS-> APP。数据分析人员一般会在 APP或 DWS层上进行分析，而不会直接针对 ODS（原始数据层）进行分析。

- **没有保留原始数据**

  企业出于成本考虑，ODS层的原始数据通常只有一定时间的保存期限。

# 3.数据仓库和数据湖对比

| 名称         | 数据仓库                        | 数据湖                           |
| ------------ | ------------------------------- | -------------------------------- |
| 读写模式     | 写时模式，数据存储前定义 Schema | 读时模式，读取数据时定义 Schema  |
| 数据价值     | 提前明确                        | 无须提前明确                     |
| 存储数据类型 | 清洗后的结构化数据              | 原始数据、非结构化和半结构化数据 |
| 容量扩展成本 | 中等成本                        | 低成本                           |
| 支持功能     | 统计、报表和传统 BI分析         | 敏捷数据集成，支持编程框架       |
| 构建成本     | 重量级，时间成本高、投资规模大  | 轻量级，比较灵活，成本低         |

**数据湖的核心理念如下**：

- 存储海量的原始数据
- 支持任意的数据格式（结构化、半结构化、非结构化）
- 较强的分析和处理能力

# 4.流行的数据湖实施方案

目前业界流行的数据湖实施方案有三种：

- **Apache Hudi**

  Hudi is a rich platform to build streaming data lakes with incremental data pipelines
  on a self-managing database layer, while being optimized for lake engines and regular batch processing.

- **Apache Iceberg**

  Iceberg is a high-performance format for huge analytic tables. Iceberg brings the reliability and simplicity of SQL tables to big data, while making it possible for engines like Spark, Trino, Flink, Presto, and Hive to safely work with the same tables, at the same time.

- **Delta Lake**

  Delta Lake is an open-source storage framework that enables building a
  [Lakehouse architecture](http://cidrdb.org/cidr2021/papers/cidr2021_paper17.pdf) with compute engines including Spark, PrestoDB, Flink, Trino, and Hive and APIs for Scala, Java, Rust, Ruby, and Python.

**下面是对三种方案的一些对比**：

| Apache Hudi              | Apache Iceberg             | Delta Lake               |
| ------------------------ | -------------------------- | ------------------------ |
| 廉价存储                 | 廉价存储                   | 廉价存储                 |
| -                        | **企业级性能**             | -                        |
| Table Schema             | Table Schema               | Table Schema             |
| -                        | **抽象程度高，不绑定引擎** | -                        |
| ACID语义保证，多版本保证 | ACID语义保证，多版本保证   | ACID语义保证，多版本保证 |
| **快速 CRUD和 Pull增量** | -                          | -                        |
| 支持流批读写             | -                          | 支持流批读写             |
| -                        | 支持 Python接口            | 支持 Python接口          |

# 5.总结

本文通过对比数据湖和数据仓库的各项特性，发现 数据湖的出现并不是为了替代数据仓库，而是**补齐数仓缺失的一些能力**。





