# 1.前言

先讲一下做这件事的背景，笔者用 Docker搭了一套 Kafka的环境用于测试，某天发现 Kafka频繁重启，查看日志中的报错如下：

```
ERROR Fatal error during KafkaServer startup. Prepare to shutdown (kafka.server.KafkaServer)
kafka.common.InconsistentClusterIdException: The Cluster ID q3r3fhGkTya24-s3dfvYUQ doesn't match stored clusterId Some(kguWHlzQQGmCHczV3u38vQ) in meta.properties. The broker is trying to join the wrong cluster. Configured zookeeper.connect may be wrong.
        at kafka.server.KafkaServer.startup(KafkaServer.scala:218)
        at kafka.Kafka$.main(Kafka.scala:109)
        at kafka.Kafka.main(Kafka.scala)
```

这里的报错很清晰：

​	Kafka **meta.properties文件中记录的 Cluster ID与 ZK中的不一致**

查看网上的资料，基本都讲的是修改 meta.properties文件中的 Cluster ID与 ZK一致即可，但由于笔者的 Kafka此时正在不断重启，完全无法进入容器内部，那么该如何去修改这个文件呢？

——**暂时找不到方法修改 meta.properties文件**

那么换个思路，是不是可以**修改 ZK中记录的 Cluster ID呢**？

# 2.如何找到 Kafka Cluster ID的存储位置

第一步，进入 ZK命令行

```shell
# 进到 ZK的安装目录后执行下面的脚本
./bin/zkCli.sh
```

第二步，使用 ls命令寻找保存 cluster id的文件

```shell
ls /kafka/cluster
# [id]
```

第三步，获取上一步找到的 id文件中的内容

```shell
get /kafka/cluster/id
# {"version":"1","id":"q3r3fhGkTya24-s3dfvYUQ"}
```

至此，已经成功找到了 ZK上存储 cluster id的文件

# 3.修改 Kafka Cluster ID

把日志里面 Kafka存储的 cluster id设置到 ZK里即可：

```shell
# 修改 cluster id
set /kafka/cluster/id {"version":"1","id":"kguWHlzQQGmCHczV3u38vQ"}

# 确认修改结果
get /kafka/cluster/id
# {"version":"1","id":"kguWHlzQQGmCHczV3u38vQ"}
```

修改完毕后重启 Kafka容器，可以看到 Kafka已经可以正常启动

# 4.总结

回溯 Kafka中存储的 cluster id和 ZK不一致的原因，应该是笔者重新创建 ZK容器导致的。

另外，解决问题的时候思路一定要打开，一条路走不通的时候，及时换一条路。





