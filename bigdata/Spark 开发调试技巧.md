# **Spark 部署模式简介：**

- **Local**

  一般就是跑在自己的本地开发机上，用于跑单元测试、学习算子的使用方式等。

- **Cluster**

  - **Standalone**

     spark 自己负责资源的管理调度。 

  - **Mesos**

     使用 mesos 来管理资源调度。

  - **Yarn**

     使用 yarn 来管理资源调度 

# **开发和调试技巧**

下面介绍的开发和调试技巧都是基于 **Spark On Yarn** 这种部署模式，这是现在企业常见的部署方式。

## 1.常用算子

spark 的算子分为2大类：**Transformations** 和 **Actions** ，spark 操作是**懒执行**的，执行**转换类算子**时不会立即执行，只是把操作记录下来，等遇到**动作类算子**的时候就会提交作业真正开始执行。这样做的好处就是 spark 执行引擎可以智能优化转换类算子的执行流程，有些操作可以进行合并，提高执行效率。

**下面分类列举一些开发中常用的算子（完整的算子介绍可以去看官方指导文档，文末有链接）：**

### 1.1 **flatMap**

**map 类转换算子**，遍历 rdd 对里面单个对象进行处理，返回 0个或多个处理后的对象。

与 map 算子相比：

​	map 算子只能是进去一个对象，返回一个对象，仅有遍历功能，比较局限。

​	flatMap 算子不仅有遍历和过滤（返回 0个对象）功能，还支持返回多个对象。

**代码示例：**

```java
// 调用方式
Rdd.flatMapToPair(new OriginPairFlatMap());

// 算子具体实现，覆盖 call 方法，在里面写自己的业务逻辑
public class OriginPairFlatMap
    implements PairFlatMapFunction<Row, String, Object> {
    
  @Override
  public Iterator<Tuple2<String, Object>> call(Row row) {
    List<Tuple2<String, Object>> results = Lists.newArrayList();
      
    // 写自己的业务逻辑 ...

    return results.iterator();
  }
}
```

### 1.2 **combineByKey**

**聚合类转换算子**，会产生 shuffle，用于把 rdd 中的数据按 key 进行聚合。

与 groupByKey 比较：

​	combineByKey 可以自定义聚合逻辑，控制聚合后的集合大小，避免 OOM；可以在本地就进行合并，效率较高。

**代码示例：**

```java
// 函数调用
JavaPairRDD<String, List<Person>> combinedRDD =
  javaPairRDD.combineByKey(
    CombineByKeyFunction.createCombinerFunction(),
    CombineByKeyFunction.createMergeValueFunction(groupThreshold),
    CombineByKeyFunction.createMergecombinerFunction(groupThreshold));

public class CombineByKeyFunction {

  // 第一个函数，创建集合，即第一次碰到这个 key 的数据时
  public static Function<Person, List<Person>> createCombinerFunction() {
    return value -> Lists.newArrayList(value);
  }

  // 第二个函数，把再次碰到的这个 key 的数据放到已有的集合里
  public static Function2<List<Person>, Person, List<Person>> createMergeValueFunction(
      int threshold) {
    return (lstValues, value) -> {
      // 控制聚合后的集合大小，避免 OOM
      if (lstValues.size() < threshold) {
        lstValues.add(value);
      }
      return lstValues;
    };
  }

  // 第三个函数，合并属于同一个 key ，但是在不同分区中的数据
  public static Function2<List<Person>, List<Person>, List<Person>> createMergecombinerFunction(
      int threshold) {
    return (allvalues, values) -> {
      if (allvalues.size() + values.size() <= threshold) {
        allvalues.addAll(values);
      }
      return allvalues;
    };
  }
}
```

### 1.3 **filter**

**转换类算子**，把 rdd 中符合要求的数据取出生成一个新的数据集。

使用 filter 算子后可能导致数据倾斜的问题，可以用 repartition 算子解决。

**代码示例：**

```java
JavaRDD<String> originRdd=sparkContext.parallelize(Arrays.asList("it","is","my","life"));
JavaRDD<String> filterRdd=originRdd.filter(word->word.length()==2);
```

### 1.4 **union**

**转换类算子**，比较简单，把 2个数据集合并。比如从 hbase 的多张表里取出了数据，合并后再处理。

**代码示例：**

```java
rdd1.union(rdd2);
```

### 1.5 **distinct**

**转换类算子**，对 rdd 进行去重。数据重复是个很普遍的现象，建议在读取原始数据后都调用 distinct 算子。

**代码示例：**

```java
rdd.distinct()
```

### 1.6 **repartition**

**转换类算子**，用于对数据进行重新分区。

**什么时候需要重新分区？**

当分区个数和数据量不匹配时。partition 和 task 是一对一的关系，如果数据量过大而分区数过少，程序产生的任务个数就少了，并行度不够，无法充分利用集群的资源，影响运行速度。反之，数据量过少而分区数过多，可能程序执行的大部分时间都是在创建任务，而执行任务的时间却很短，这就反客为主了。

**代码示例：**

```java
rdd.repartition(500);
```

### 1.7 **saveAsTextFile** 

**动作类算子**，一般把计算好的结果存到 hdfs 上供后续使用。

**代码示例：**

```java
rdd.saveAsTextFile("/xxx/yyy");
```

## 2.如何设置程序执行参数

先看一个完整的程序提交命令：

```shell
/usr/hdp/current/spark/bin/spark-submit --master yarn 
--conf spark.local.dir=/opt/tmp 
--driver-memory 2g 
--executor-memory 6G 
--conf spark.yarn.executor.memoryOverhead=2048 
--conf spark.executor.cores=3 
--conf spark.scheduler.mode=FAIR 
--conf spark.yarn.max.executor.failures=1024 
--conf spark.dynamicAllocation.enabled=true 
--conf spark.dynamicAllocation.minExecutors=2 
--conf spark.shuffle.service.enabled=true 
--queue testqueue 
--conf spark.sql.hive.convertMetastoreOrc=false 
--conf spark.sql.crossJoin.enabled=true 
--files file1 
--class com.lzy.daphnis.service.DaphnisAttributeStat /opt/test-spark/lib/daphnis-*-jar-with-dependencies.jar 
```

**spark-submit --master yarn** 指定程序执行模式为 yarn 。

**spark.yarn.executor.memoryOverhead** 这个参数是和 --executor-memory 一起产生作用的，**比较重要**，如果不设或者设得比较小，spark 程序很容易失败。原因如下：

​	--executor-memory 很容易理解，就是每个 executor 分配的内存，但是实际发现，程序在运行时很容易超过这个内存值，然后 executor 就被 shutdown 了，task 失败。memoryOverhead 就是为了解决这个情况，它允许executor 使用的内存超过设置的值一部分，这样一来 task 成功率就高了很多。

**spark.dynamicAllocation.enabled** 是否动态分配资源，可以作为一般配置，有 yarn 去自动分配 exexutor 资源，建议是开启的。后面还可以用别的参数调整并行度，更充分的利用集群资源。

**--queue** 这个和它的含义一样，指定程序运行使用的队列，程序运行的资源限制决定于队列被分配的资源。

**--files** 指定需要上传到集群的文件，如果执行的是 yarn cluster 模式，driver 不是当前机器，是无法读取到本地文件的，此时就需要这个参数上传到集群才能读取。

**--class** 指定程序运行的入口和 lib 包。

这里没有指定程序提交方式，就是默认的 yarn client。一般常用的程序提交参数就这些，有时候会根据程序业务逻辑不同再添加一些参数，比如设置默认的并行度、序列化方式等。

## 3.选择程序的提交模式

- **client 模式**

  这个模式一般用于**调试**或者需要直接在程序日志里面拿到**详细的程序运行信息**。

  client 模式会回选取当前提交程序的机器作为 driver ，也就是当前执行 spark 程序的这台机器。

  如何设置程序的提交模式 client 模式：

  ```shell
  ... --master yarn --deploy-mode client ...
  ```

- **cluster 模式**

  这个模式一般用于**生产环境**，程序里只能拿到简单的执行信息，这个时候的程序日志基本没有什么用。如果程序出现问题，基本上只能把 executor 里的日志拿下来进行分析。

  cluster 模式会选取集群中的某一台机器成为 driver 。

  如何设置程序的提交模式为 cluster 模式：

  ```shell
  ... --master yarn --deploy-mode cluster ...
  ```

## 4.调试技巧

### 4.1 使用  **collect** 算子打印中间结果

有时调试问题需要打印程序的中间结果，可以使用 collect 算子把数据收集到 driver 端，然后打印到日志，如下：

```java
List<String> tmpData=rdd.collect()
....
```

如果只需要取部分元素，也可以用 take 。

### 4.2 强制杀掉 spark 程序

先找到这个程序的 application id，程序日志里一般就有，或者从 hadoop 管理界面去找，然后执行命令：

```shell
yarn application -kill <appid>
```

### 4.3 收集所有 executor 的日志到本地

executor 里面记录了更详细的程序执行信息，甚至有些报错只能在 executor 日志里找到，因此学会这个技能很重要。如下：

```shell
yarn logs -applicationId <appid> > executors.log
```

### 4.4 观察程序执行情况

写出一个 spark 程序其实不难，熟悉了各个算子后，进行一些组合就能实现业务逻辑，但是程序能否抗住大数据量的检验，这个需要做的事情就比较多了。

即使使用 yarn client 模式，在日志中能看到的信息也比较有限。其实 hadoop 有提供 spark 程序的运行管理界面，一般是集群 **ResourceManager** 的  IP 加上 **8088** 端口就可以打开管理界面。当然，某些大数据平台厂商会修改这个端口号，但一般能在集群管理界面找到 RM 管理界面的入口。

打开程序管理界面后，注意观察 **Stage** 和 **Executor** 这两个界面，可以找到程序执行慢的地方，卡住或者崩溃的地方。观察 executor 的数量也能知道自己程序的并行度够不够。

### 4.5 打印额外的程序信息

在提交程序的参数里面加上这个参数：

```shell
--verbose
```

可以打印出 spark 程序执行的一些额外信息，比如依赖的 jar 包路径，曾经利用这个参数解决了一个很难弄的 jar 包冲突的问题。

## 5.常见问题和解决方案

### 5.1 OOM 原因分析和解决

科普下 OOM 就是常见的 **OutOfMemoryError** 错误。

那么 spark 程序在什么情况下容易产生 OOM 呢？

- **mapPartition 算子**

  map 算子的加强版吧，以分区为单位遍历 rdd 中的数据。但是当某个分区数据特别多的时候，就极易产生 OOM 。

  解决办法：可以换成 map 算子，牺牲一些性能，但是程序能跑过；或者解决数据倾斜的问题，保证没有哪个分区数据量会特别大。

- **根据 key 进行分组的算子**

  比如 groupByKey，combineByKey。

  在对数据进行分组时，会产生 shuffle ，如果某个分区流入过多的数据，自然就 OOM 了。

  groupByKey 只能是解决数据不均衡的问题，而 combineByKey 可以控制分区中的数据量，也就可以避免 OOM 了。

- **spark sql 执行 join 操作**

  这个一般也是 shuffle 导致的 OOM 。

  解决的话，如果是大小表，可以把小表的数据广播到 executor ，在 map 的时候做 join。否则可以尝试调大 **spark.sql.shuffle.partitions** 参数，增加程序的并行度。

- **没有正确设置  memoryOverhead 参数**

  spark 程序在执行时很容易达到 executor 的内存限制，memoryOverhead 没设置或者设得比较小，就直接 OOM  了。memoryOverhead 可以设置 executor 使用一部分额外的内存，设置得稍微大一些，就不容易出现 OOM 了。

### 5.2 SOF 原因分析和解决

科普下 SOF 就是常见的 **StackOverFlowError** 错误。spark 程序报这个错误的原因和 java 程序是一样的，调用链过长导致栈内存不足，进而引发异常。

那么 spark 又是如何出现调用链过长的呢？

- **程序里面有很多的转换类算子**

  spark 是懒执行的，会先把操作记录下来，如果一直碰不到动作类算子，就有可能导致调用链过长，进而 SOF。

  解决：可以在中间调用动作算子 persist 或者 checkpoint 触发计算。

- **在大表上执行复杂的 spark sql**

  为什么是大表，spark 调用链的长度似乎跟数据量是有关系的。同一条 sql ，小表没有问题，大表就会 SOF ，这个具体原因，后面再详细研究下。

  解决：最直接的就是优化 sql  ；其次把 sql 中需要做的部分操作移到 spark 程序中去做。

### 5.3 Spark 程序执行慢

决定 spark 程序执行速度的核心因素是**数据量**和**并行度**。比如有 500亿数据，用 3台机器的集群去处理，这个无论如何快不起来的。

这里提供几个程序执行慢的具体排查解决思路：

- **是否是数据量过大，集群处理能力不足**

  这种情况，建议分批处理数据，否则很可能程序跑不过。

- **是不是自己的程序并行度不够，没有充分利用集群的资源**

  这个可以观察集群的资源占用情况，如果一直很低就有问题了。

  可以在程序提交参数里面设置默认并行度。

- **是不是程序所使用的队列资源分配有问题**

  这个可能是集群管理员给队列分配的资源少了，加资源就可以了。

- **是不是程序中有多处 shuffle 操作**

  shuffle 操作很耗时间和内存，当内存不够时，还会溢写磁盘，这个就更慢了。

  尽量简化程序，去掉不必要的 shuffle 操作。另外还有一些针对 shuffle 调优的参数可以试一下。

- **是不是存在大量 executor 挂掉，真实执行任务的只有几个 executor 的情况**

  程序写得有问题，最后就剩几个 executor 在工作，自然就慢了。





# 参考资料

1. [Spark 官方文档-算子介绍](https://spark.apache.org/docs/2.1.0/programming-guide.html#transformations)
2. [Spark 官方文档-yarn 参数](http://spark.apache.org/docs/2.1.0/running-on-yarn.html#configuration)
3. [spark task、job、partition之间的关系 宽窄依赖 spark任务调度](https://blog.csdn.net/qq_41950069/article/details/80828862)