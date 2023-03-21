# 1.前言

目前，很多 flink相关的书籍和网上的文章讲解如何对接 kafka时都是使用的 FlinkKafkaConsumer，如下：

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

Properties properties = new Properties();
//指定kafka的Broker地址
properties.setProperty("bootstrap.servers", "192.168.xx.xx:9092");
//指定组ID
properties.setProperty("group.id", "flink-demo");
//如果没有记录偏移量，第一次从最开始消费
properties.setProperty("auto.offset.reset", "earliest");

FlinkKafkaConsumer<String> kafkaSource = new FlinkKafkaConsumer<>("topic005", new SimpleStringSchema(), properties);
```

新版的 flink，比如 1.14.3已经将 FlinkKafkaConsumer标记为 deprecated（不推荐）,如下：

```
'org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer' is deprecated 
```

# 2.如何使用 KafkaSource

新版本的 flink应该使用 KafkaSource来消费 kafka中的数据，详细代码如下：

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
KafkaSource<String> kfkSource = KafkaSource.<String>builder()
    .setBootstrapServers("192.168.xx.xx:9092")
    .setGroupId("flink-demo")
    .setTopics("topic005")
    .setStartingOffsets(OffsetsInitializer.earliest())
    .setValueOnlyDeserializer(new SimpleStringSchema())
    .build();
DataStreamSource<String> lines = env.fromSource(kfkSource, WatermarkStrategy.noWatermarks(), "kafka source");
```

# 3.总结

开发者在工作中应该尽量避免使用被标记为 deprecated的方法或者类，要及时进行更换，保持代码的活力。

