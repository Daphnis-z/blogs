# 1. 简介

Redis 是一个开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存、分布式、可选持久性的键值对(**Key-Value**)存储数据库，并提供多种语言的 API。

**Redis 特点：**

- 支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用
- 不仅仅支持简单的 key-value类型的数据，同时还提供 list，set，zset，hash等数据结构的存储
- 支持数据的备份，即 master-slave 模式的数据备份

**Redis 优势：**

- **性能极高**

  读速度最快能达到 110000次/s,写速度最快能达到 81000次/s 

- **数据类型丰富**

  支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作

- **原子性**

  所有操作都是原子性的，要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过 MULTI和 EXEC指令包起来

- **丰富的特性**

  持 publish/subscribe, 通知, key 过期等等特性

Redis 默认支持 **16个**数据库，数据库以一个从 0开始的递增数字命名。

# 2. 安装运行

这里以借助 **docker** 来安装和运行 redis：

 1. 安装

    ```
    docker pull redis
    ```

 2. 配置

    这里主要开启 redis 远程连接，设置密码

    ```
    # 创建所需文件夹
    mkdir -p /opt/docker/redis/data
    mkdir -p /opt/docker/redis/conf
    
    # 添加所需配置
    vim /opt/docker/redis/conf/redis.conf
    # 需要添加的内容如下
    #bind 127.0.0.1
    protected-mode no
    appendonly yes
    requirepass daphnis
    ```

 3. 启动

    ```
    # 创建 Redis 容器并使之后台运行
    docker run --name redis101 -p 6379:6379 -v /opt/docker/redis/data:/data -v /opt/docker/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf
    ```

 4. 验证

    ```
    # 无密码进入 redis 命令行
    # --raw 能防止中文乱码
    docker exec -it <container-id|container-name> redis-cli --raw
    # eg. docker exec -it redis101 redis-cli --raw
    ```

# 3. 基本数据类型

## 3.1 String

redis 最基本的数据类型，最大能存储 512M

```
# 添加 key-value
set hello redis

# 获取 key 的值
get hello
```

**使用：**

​	可以将任何二进制文件转为字符串后进行存储

## 3.2 Hash

key-value 集合。

```
# 添加值
hset hash-test field5 20

# 获取值
hget hash-test field5
```

**使用：**

​	适合存储对象的各个属性，或者一组相关联的 key-value

## 3.3 List

简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部（左边）或者尾部（右边）

```
# 往列表左边添加一个值
lpush mobile 5

# 往列表右边添加一个值
rpush mobile 2

# 取出列表中的部分元素
lrange mobile 0 1
```

**使用：**

​	双向链表，增删快，可以批量操作元素，可以用作小型消息队列

## 3.4 Set

string 类型的**无序不重复集合**。通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)

```
# 添加元素
sadd set-test hello

# 查看元素
smembers set-test
```

**使用：**

​	存储一些相关元素，并进行去重；进行一些集合间的运算，如 求差集、交集、并集

## 3.5 Zset

string 类型的**有序不重复集合**。

每个元素都会关联一个 double类型的分数，使用分数来为集合中的成员进行从小到大的排序

```
# 添加元素
zadd zset-test 0.6 hello
 
# 取出指定分数段的元素
zrangebyscore zset-test 0.5 0.7
```

**使用：**

​	根据分数筛选符合要求的元素集合

# 4. 特性

## 4.1 发布订阅

​	一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。客户端可以订阅任意数量的 channel，当 channel 收到消息后会主动将消息推动给订阅过它的客户端。

**推模式** 的消息队列。

**缺点：**

​	**消息无法持久化**，如果出现网络断开、Redis 宕机等，消息就会被丢弃。

## 4.2 Stream

Redis Stream 主要用于消息队列（Message Queue）。

Stream 提供了消息的**持久化**和**主备复制**功能，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。

**拉模式**的消息队列。

## 4.3 事务

​	事务由命令 **multi 开始**，**exec 结束**，在这两个命令间输入需要执行的若干个命令。注意 单个命令具有原子性，而事务不具有原子性。

**三个保证：**

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

**三个阶段：**

- 开始事务。
- 命令入队。
- 执行事务。

## 4.4 脚本

​	Redis 脚本使用 Lua 解释器来执行脚本，执行脚本的命令是 eval。

# 5. 实用命令

## 5.1 查看配置

```
# 查看所有配置
config get *

# 查看支持的数据库数量
config get databases
```

## 5.2 连接命令

```
# 验证密码
auth <password>
# eg. auth 123456

# 查看服务是否运行
ping

# 关闭当前连接
quit

# 切换数据库
select <num>
# eg. select 5
```

## 5.3 字符串命令

```
# 获取 key 对应 value 的长度
strlen key

# key 对应的 value +1
incr key

# key 对应的 value -1
decr key
```



TODO:

- 客户端侧缓存
- 分布式锁——RedLock

​	



# 参考

1. [Redis Documentation](https://redis.io/documentation)
2. [菜鸟 Redis教程](https://www.runoob.com/redis/redis-tutorial.html)







