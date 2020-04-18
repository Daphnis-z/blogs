

## 出现重复消费的**根本原因**： 

​	**客户端已经消费了数据，但是 offset 没有提交**。

## offset 没有提交的原因一般有 3种：

- **设置了自动提交 offset ，由于程序的异常导致了漏提交**
- **consumer 被 group coordinator 从当前消费组中移除**
- **consumer 提交 offset 失败**

备注：group coordinator 可以理解为 consumer group 的协同管理者。

## 下面来具体分析这几种情况，以及如何解决：

1. **设置了自动提交 offset ，由于程序的异常导致了漏提交**

   先看下如何设置自动提交和提交的频率：

   ```properties
   # 下面等号右边的值均为官方默认值
   
   # 重要性：中
   enable.auto.commit=true
   # 重要性：低
   auto.commit.interval.ms=5000
   ```

   如果在 offset 还没提交时，程序出现异常导致了重启就会重复消费。

   **解决：**

   ​	在程序里面注意**捕获异常**和响应**进程关闭**事件（万不得已不要用 kill -9），在 finally 里面调用 **consumer.close()** 及时提交 offset。

   如果开启了自动提交，在调用 consumer.close() 方法的时候就会触发 offset 的提交，源码如下：

   ```java
   if (this.autoCommitEnabled) {
     this.autoCommitTask.disable();
   
     try {
       this.commitOffsetsSync(this.subscriptions.allConsumed());
     } catch (WakeupException var2) {
       throw var2;
     } catch (Exception var3) {
       log.warn("Auto offset commit failed: ", var3.getMessage());
     }
   }
   ```

2. **consumer 被 group coordinator 从当前消费组中移除**

   被**移除的原因**是 coordinator 认为当前 consumer 已经挂了。

   这里需要提到两个参数：

   ```properties
   # 下面等号右边的值均为官方默认值
   
   # 重要性：高
   session.timeout.ms=10000
   # 重要性：高
   heartbeat.interval.ms=3000
   # 这两个参数如何设置，官方建议 heartbeat.interval 小于 session.timeout 的 1/3
   ```

   如果 coordinator 在 session.timeout.ms 参数指定的时间内没有收到任何 consumer 的消息就会认定该 consumer 挂掉了，从而将其移出消费组，触发 rebalance 。consumer 这边其实有2个线程，工作线程和心跳线程，心跳线程会每隔 heartbeat.interval.ms 指定的时间向 coordinator 发送一条消息证明当前 consumer 还活着。

   consumer 被移除后自然无法提交 offset ，由于触发了 rebalance ，当前消费的 partition 可能被分配给别的 consumer 造成重复消费。

   **解决：**

   ​	有时候 consumer 可能只是短暂的出现了问题，所以可以适当调大 session.timeout.ms 的值。

3. **consumer 提交 offset 失败**

   提交失败的常见原因是什么呢？

   先解释下 **max.poll.interval.ms（默认 300秒）** 参数：

   > The maximum delay between invocations of poll() when using consumer group management. This places an upper bound on the amount of time that the consumer can be idle before fetching more records. If poll() is not called before expiration of this timeout, then the consumer is considered failed and the group will rebalance in order to reassign the partitions to another member. For consumers using a non-null `group.instance.id` which reach this timeout, partitions will not be immediately reassigned. Instead, the consumer will stop sending heartbeats and partitions will be reassigned after expiration of `session.timeout.ms`. This mirrors the behavior of a static consumer which has shutdown.

   如果在 poll 完数据后，长时间都在处理这批数据，等到下一次 poll 时已经超过了 max.poll.interval.ms 指定的时间，提交 offset 就会失败，当前消费的 partition 就会被重新分配。
   **解决：**
   ​		首先应该想到的是程序处理逻辑是不是有问题，需要花这么长时间才能处理完这批数据；
    ​		其次，程序逻辑没问题的情况下可以调大 max.poll.interval.ms 的值，或者减少 **max.poll.records（单次 poll 到的数据量，默认 500条）** 的值。

   

## 贴一下官方文档：

​	[kafka consumer 参数解释](http://kafka.apache.org/documentation/#consumerconfigs)
