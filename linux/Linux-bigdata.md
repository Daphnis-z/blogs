### 1. linux

```shell
# 查看当前文件夹下各个目录大小
du -sh *
# 查看指定目录空间使用情况
df -h <dir>
# 每一个文件占用一个 inode ,inode 记录了文件的权限属性等信息
# 查看 inode 使用情况
df -i

# 去到上一次的目录
cd -

# 按时间降序排列显示当前文件夹下的文件
ll -t
# 统计当前目录下（含子目录）文件个数
ls -lR |grep "^-"|wc -l

# 修改 dns 服务器名称
vim /etc/resolv.conf

# centos7 永久修改主机名
vim /etc/hostname

# 远程复制文件
# -r 递归复制
#           源路径       目标路径
scp -r <user>@ip:<path> <path>

# centos 查看系统版本
cat /etc/redhat-release

# 防火墙
# 检查selinux
/usr/sbin/sestatus -v

# grep
# 完全匹配
grep -w

# 文件权限
# 修改文件为 可读写
chmod -R 777 <file>

# echo
# echo 输出不同颜色的消息
echo -e "\033[30m 黑色字 \033[0m"
echo -e "\033[31m 红色字 \033[0m" 
echo -e "\033[32m 绿色字 \033[0m" 
echo -e "\033[33m 黄色字 \033[0m" 
echo -e "\033[34m 蓝色字 \033[0m" 
echo -e "\033[35m 紫色字 \033[0m" 
echo -e "\033[36m 天蓝字 \033[0m" 
echo -e "\033[37m 白色字 \033[0m"

# 永久修改主机名称 hostname
# 改完之后要重启机器
# centos6
vim /etc/sysconfig/network
# centos7
vim /etc/hostname

# centos7 设置静态 ip
# 第一步：更改文件 /etc/sysconfig/network-scripts/ifcfg-ens33 的内容:
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static  #静态IP 
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
DEVICE=ens33 
NAME=ens33 
UUID=2ef0c6db-10b0-4cf8-99ee-3bed59c8a768   # 替换下
NM_CONTROLLED=no      #表示该接口将通过该配置文件进行设置，而不是通过网络管理器进行管理
ONBOOT=yes  #开机启动
IPADDR=192.168.11.56   #本机地址
NETMASK=255.255.255.0  #子网掩码
GATEWAY=192.168.11.1    #默认网关，注意如果修改过网络设置，这里也需要改下
# 第二步：修改文件 /etc/sysconfig/network的内容
NETWORKING=yes
DNS1=10.1.12.4
# 第三步：重启网络服务
systemctl restart network

# 获取上周日的日期
date -d "last sunday" +%F

# 如何获取上个月最后一个周日的日期
current_month_first_day=`date +%Y%m01`
last_month_end_day=`date -d "$current_month_first_day last day" +%F`
echo "last_month_end_day: $last_month_end_day"
week_n=`date -d "$last_month_end_day" +%w`
last_month_end_sun=`date -d "$last_month_end_day $week_n days ago" +%F`
echo "last_month_end_sun: $last_month_end_sun"

# 查看端口号占用情况
lsof -i:9095

# 查看帮助信息
man <cmd>
info <cmd>
# 查看命令的位置
which <cmd>

# 查看网络信息
netstat -a

# hello.txt
basename /opt/hello.txt
# /opt
dirname /opt/hello.txt

# 定义别名
alias cls='clear'
# 取消别名
unalias cls

# 垃圾桶，不需要显示的信息可以往这里丢
/dev/null

# 双向重导向，可以把脚本执行的信息同时输出到屏幕和日志文件
tee -a test.log

# 分割文件
split

# 查看系统的开机信息 
dmesg
```

#### 1.1 process

```shell
# 查找进程
ps -ef|grep <text>
# 强制杀掉进程
kill -9 <process id>
# 杀掉含有关键字 text 的进程
ps -ef|grep [<text>]|grep -v grep|awk  '{print "kill -9 " $2}' |sh
```

#### 1.2 service

```shell
# centos6
状态： service <service name> status
启动： service <service name> start
重启： service <service name> restart
```

#### 1.3 compress

```shell
# tar.gz
压缩： tar -zcvf <file>.tar.gz <dir>
解压： tar -zxvf <file>.tar.gz -C <dir>

# tar
解压： tar xvf <file>.tar -C <dir>

# gz
解压： gunzip ***.gz

# zip
压缩： zip -r <file>.zip <dir>
解压： unzip <file>.zip
```

#### 1.4 ch

​	-rwxr-xr--. 1 root root 0 Dec  8 04:02 test.log

​	解释：- 表示这是一个文件，rwx 表示文件拥有者的权限，r-x 表示文件所属组的权限，r-- 表示其他账户的权限

- chgrp: 修改文件所属群组

- chown: 修改文件拥有者

- chmod: 修改文件权限

  ```
  # r:4,w:2,x:1 rwx=4+2+1=7
  # 修改文件为可读写、执行：
  chmod 777 <file>
  ```

#### 1.5 grep

```shell
# grep
# 完全匹配
grep -w

# 保留含有 root 或 dbus 的行
ps aux |egrep '(root|dbus)'
```

#### 1.6 vim

```shell
# 搜索文本文件里面的关键词
# 向前搜索
/word
# 向后搜索
?word
# 向前或向后移动
n/N

# 在 vim 编辑器中批量替换，word1 替换为 word2
:1,$s/word1/word2/g

# 复制整行
yy
# 区块复制
ctrl+v 选中复制区域，y 进行复制 p 进行粘贴

# 撤销上一次操作
u
# 重做上一个动作
ctrl+r

# 保存修改
:w
# 强制保存修改
:w!

# 设置行号
:set nu
# 取消行号
:set nonu

# 移动到文件末尾
G
```

### 2. text

```shell
# 合并文本文件
cat files/* > file_merge

# 合并两个文本文件，重复的行只保留一份
cat file1 file2 |sort |uniq > <file>

# 替换文本文件中某些字符 eg.把'\'替换成'\t'
sed -i 's/\\/\t/g' <file>

# 删除特定文本所在行
sed -i '/000000000000000000/d' <file>

# 在文件中查找文本
# * : 表示当前目录所有文件，也可以是某个文件名
# -r 是递归查找
# -n 是显示行号
# -R 查找所有文件包含子目录
# -i 忽略大小写
grep -rn "hello,world!" *

# 查看文本文件行数
wc -l <file>
```

### 3. rpm

```shell
# 安装一个包 
rpm -ivh [name]

# 卸载一个包 
rpm -e 

# 升级一个包 
rpm -Uvh 

# 查找已安装的rpm包
rpm -qa |grep dolphin

# 列出服务器上的一个文件属于哪一个RPM包 
rpm -qf 

7.列出该包中有哪些文件 
# rpm -ql [rpm package name]

# 列出一个未被安装进系统的RPM包文件中包含有哪些文件？ 
rpm -qilp [rpm package name]

# 解压RPM包
rpm2cpio xxx.rpm | cpio -div
```

### 4. yum

```shell
# 查看yum源
yum repolist

# 查看可用的软件包
yum list |grep mysql

# 查看已安装的包
yum list installed |grep mysql
```

### 6. bigdata

#### 6.1 spark

```shell
# 杀掉yarn任务
yarn application -kill <appid>

# 切换yarn模式
# yarn-client
--master yarn --deploy-mode client
# yarn-cluster
--master yarn --deploy-mode cluster

# 收集executor日志
yarn logs -applicationId <appid> > <file>

# 查看 spark 版本
spark-submit --version
```

Spark改为yarn-cluster模式后报文件找不到:

​	使用Spark指令：--files file1,file2..  将所需文件上传，文件不能带路径，只能是文件名。

#### 6.2 hive

```shell
# 显示列名
set hive.cli.print.header=true;

# 设置执行引擎
# mr
set hive.execution.engine=mr;
# tez
set hive.execution.engine=tez;

# 查看表空间下的 orc 文件
hdfs dfs -ls /apps/hive/warehouse/simba.db/simba_phy_attribute

# 含有分区列的表批量插入数据
set hive.exec.dynamic.partition.mode=nonstrict;
insert into table simba_phy_attribute_tmp PARTITION(action_time,import_time) select uuid,code_type,code,attribute_type,attribute,code_area,attribute_area,capture_time,longitude,latitude,location_type,location,location_area,action,origin_data,action_time,import_time from simba_phy_attribute where length(trim(code))>0 and length(trim(attribute))>0;
```

#### 6.3 hadoop

```shell
# 列出某个路径下的所有文件
hadoop fs -ls <path>

# 查看某个文件内容
hadoop fs -cat <file>
```

#### 6.4 hbase

```shell
# 进入hbase命令行
hbase shell

# 查看所有的表
list

# 查看某张表的所有数据
scan 'tablename'
# 查看部分数据
scan 'leo:data',{LIMIT=>10}

# 获取某行的值
get 'tablename','rowkey'

# 往某行某列插入数据
put 'tablename','rowkey','columnkey','value'

# 删除某行数据
deleteall 'tablename','rowkey'
# 清空表数据
truncate 'dolphin:roam_data'

# 查看连接hbase的zookeeper地址和端口号
vim /etc/hbase/conf/hbase-site.xml

# 创建namespace
create_namespace 'leo'
# 建表
create 'dolphin:space_time_data','point'

# 查看类命令
# 查看所有的命名空间
list_namespace
# 查看某个命名空间下所有的表
list_namespace_tables 'lob'
# 查看所有的表
list

# 查看客户端里面的中文
python
print '\xE5\x8D\x97\xE4\xBA\xAC\xE4\xB8\x8B\xE5\x85\xB3\xE5\x8C\xBA'.decode('utf-8')
```

在 spark 或普通程序里面操作 hbase 出现无限阻塞的情况，排查思路：

1. 检查认证，spark 程序要注意是否打开了 hbase 的认证

   检查认证要注意机器和集群对时，相应账户权限是否够。

   开启 hbase 认证：

   vim spark/conf/spark-defaults.conf

   ```
   spark.yarn.security.credentials.hbase.enabled=true 
   ```

2. 有时并不是是无限阻塞，timeout 了就会抛出错误

   注意不要提前 kill 程序，否则可能看不到关键性报错。

   减少 timeout 时间可以修改 hbase-site.xml 里面的 hbase.rpc.timeout。

3. 检查集群状态和 RegionServer

   如果有问题并且条件允许，可以先试着重启 hbase 的组件，否则去查看 hbase 相关组件的日志，解决问题。

4. spark 和 hbase 在不同集群且开启了安全机制，注意是否开启了集群互信

   两个集群间所有机器 hosts 文件需要把主机名称互相加进去，否则连接 RegionServer 会有问题。

   真实场景中，华为平台出现过这个问题。

#### 6.5 hdfs

```shell
# 创建文件夹
hdfs dfs -mkdir /leo_analysis/relation

# 查看hdfs各个目录占用空间大小
hdfs dfs -du -h /user/lzy/dolphin

# 上传文件到hdfs，支持通配符
hdfs dfs -put /opt/leo-relation-calc/input/*.bcp /leo_analysis/relation/shanxi-hj

# 查看文件内容
hdfs dfs -cat /leo_analysis/relation/shanxi-hj/trans_shanxi-hj_20190102_115929_1_04afefcb-2115-404b-8564-211b0a0f5ba0_29039.bcp

# 删除文件
hdfs dfs -rm -f /leo_analysis/relation/shanxi-hj/*.bcp

# 查看集群存储空间使用情况
hdfs dfsadmin -report

# 查看namenode状态
hdfs haadmin -getServiceState nn2
# 手动切换namenode状态
hdfs haadmin -transitionToActive --forcemanual nn1

# 给路径赋读写权限
hdfs dfs -chmod -R 777 /user/lzy/dolphin
```

#### 6.6 kafka

```
# 查看某台机器上的topic列表
# 腾讯
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper tbds-10-1-50-21:2181
# ambari
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper ambari-10-1-50-40:2181

# 查看 topic 详细情况
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --zookeeper ambari-10-1-50-40:2181,ambari-10-1-50-41:2181,ambari-10-1-50-42:2181 --describe --topic subscibe_dolphin

# 创建topic
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --zookeeper tbds-10-1-50-21:2181 --replication-factor 1 --partitions 1 --topic zzdaphnis

# 生产者——腾讯
/usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list tbds-10-1-50-22:6667,tbds-10-1-50-23:6667,tbds-10-1-50-24:6667 --topic result_shitifenxi --producer.config /usr/hdp/current/kafka-broker/config/client_plain.properties
# 生产者——ambari
/usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list 10.1.50.40:6667 --topic result_dc_cdr --producer.config /usr/hdp/current/kafka-broker/config/producer.properties

# 消费者——腾讯
/usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server tbds-10-1-50-22:6668,tbds-10-1-50-23:6667,tbds-10-1-50-24:6668 --topic  result_shitifenxi  --from-beginning --consumer.config  /usr/hdp/current/kafka-broker/config/client_plain.properties --new-consumer
# 消费者——ambari，55.11上有客户端
/usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server 10.1.50.40:6667 --topic subscibe_dolphin --from-beginning --consumer.config /usr/hdp/current/kafka-broker/config/consumer.properties --new-consumer

# 查询某个consumer topic数据积压情况
/usr/hdp/2.2.0.0-2041/kafka/bin/kafka-consumer-groups.sh --bootstrap-server tbds-10-1-50-22:6667,tbds-10-1-50-23:6667,tbds-10-1-50-24:6667 --new-consumer  --group dataflow-hive-consumer --describe --command-config /usr/hdp/2.2.0.0-2041/kafka/config/ssl.properties
ssl.properties：
	security.protocol=SASL_PLAINTEXT
	sasl.mechanism=PLAIN
	
# kafka 日志
# 腾讯
/data/var/log/kafka
```

**华为 kafka 常用指令：**

```shell
查看当前集群Topic列表。
bin/kafka-topics.sh --list --zookeeper <ZooKeeper集群IP:24002/kafka>

查看单个Topic详细信息。
bin/kafka-topics.sh --describe --zookeeper <ZooKeeper集群IP:24002/kafka> --topic <Topic名称>

删除Topic，由管理员用户操作。
bin/kafka-topics.sh --delete --zookeeper <ZooKeeper集群IP:24002/kafka> --topic <Topic名称>

创建Topic，由管理员用户操作。
bin/kafka-topics.sh --create --zookeeper <ZooKeeper集群IP:24002/kafka> --partitions 6 --replication-factor 2 --topic <Topic名称>

Old Producer API生产数据，服务端“allow.everyone.if.no.acl.found”配置为“True”。
bin/kafka-console-producer.sh --broker-list <Kafka集群IP:21005> --topic <Topic名称> --old-producer -sync

Old Consumer API消费数据，服务端“allow.everyone.if.no.acl.found”配置为“True”。
bin/kafka-console-consumer.sh --zookeeper <ZooKeeper集群IP:24002/kafka> --topic <Topic名称> --from-beginning

赋Consumer权限命令，由管理员用户操作。
bin/kafka-acls.sh --authorizer-properties zookeeper.connect=<ZooKeeper集群IP:24002/kafka > --add --allow-principal User: <用户名> --consumer --topic <Topic名称> --group <消费者组名称>

赋Producer权限命令，由管理员用户操作。
bin/kafka-acls.sh --authorizer-properties zookeeper.connect=<ZooKeeper集群IP:24002/kafka > --add --allow-principal User: <用户名> --producer --topic <Topic名称>

New Producer API生产消息，需要拥有该Topic生产者权限。
bin/kafka-console-producer.sh --broker-list <Kafka集群IP:21007> --topic <Topic名称> --producer.config config/producer.properties

New Consumer API消费数据，需要拥有该Topic的消费者权限
bin/kafka-console-consumer.sh --topic <Topic名称> --bootstrap-server <Kafka集群IP:21007> --new-consumer --consumer.config config/consumer.properties
```

如果程序出现无法消费数据（poll 时无限阻塞），该如何排查解决：

1. 先看下能否使用命令在命令行消费某个 topic 的数据

   如果不行，证明集群有问题，去检查集群和 broker ，还有当前机器是否和集群对时。

   如果可以，需要检查代码是否有问题（比如 认证）。

2. 检查所有的 broker 看是否有异常

   通过集群管理界面看下 broker 状态即可，如果发现某个 broker 有问题，情况允许可以先重启该 broker ，如果还是不行，去看下这个 broker 的日志，看下具体报什么错。

   看日志的时候如果没有什么发现可以调整日志级别为 debug 。

3. 排查是否存在某个 topic 有问题（比如分区丢失 leader）

   这个需要去判断下哪个 topic 可能有问题，然后去检查它的所有分区。

4. 检查是不是有多个 consumer id 相同的程序在消费同一个 topic ，导致数据的争夺

如何重新消费某个 topic 里的数据？

​	比较简单，修改 consumer id 即可：group.id=dolphin-realtime-consumer

#### 6.7 cdh

```shell
# 启动代理服务
/etc/init.d/cloudera-scm-agent start

# 启动服务端数据库（postgres）
/etc/init.d/cloudera-scm-server-db start 

# 启动集群管理服务器（不启动无法访问 7180 端口，管理集群）
# 启动前要确保 scm-agent 和 scm-server-db 已启动
/etc/init.d/cloudera-scm-server start
```

### 7. oracle

```shell
# 进入Oracle
sqlplus  sys/sys as sysdba

# 启动监听器
lsnrctl start
# 查看Oracle监听器
netstat -nap | grep 1521 | grep LISTEN

# 查询所有表空间的大小
sqlplus sys/sys as sysdba
select tablespace_name, sum(bytes)/1024/1024 from dba_data_files group by tablespace_name;

# 导入导出dump文件
exp daphnis/123456 tables=DUMPTEST file=/opt/dumptest/DUMPTEST.dump
imp daphnis/123456 file=DUMPTEST.dump tables="DUMPTEST" IGNORE=Y
```

### 8.  jstack

```shell
# 查看java程序堆栈
jstack <port>
```

### 9. putty

```
# 建立隧道
1》SSH/Tunnels  Source port: 15087 , Destination: 115.15.1.87:1521
2》点 Add
3》Session 建一个会话 然后保存
4》打开连接
```

### 10. tox

```shell
# 构造 python 开发环境
tox -e devenv

# 运行 tox 测试
tox

# 运行 tox 做 release 包
tox -e py27-release
```

### 11. vagrant

```
vagrant init  # 初始化
vagrant up  # 启动虚拟机
vagrant halt  # 关闭虚拟机
vagrant reload  # 重启虚拟机
vagrant ssh  # SSH 至虚拟机
vagrant status  # 查看虚拟机运行状态
vagrant destroy  # 销毁当前虚拟机
```

### 12. ansible

```
查看自动部署脚本
cd /vagrant/auto-deploy
vim auto-deploy.sh

# 查看ansible所有task：
ansible-playbook -i environments/dev site.yml --list-tasks

# ansible从某个task开始执行：
ansible-playbook -i environments/dev site.yml --skip-tags "bind,common,sshd_config"  -v --start-at-task "lob_master_analysis : config cdh5"

# 执行某个 tag
ansible-playbook -i environments/dev site.yml --tags "leo_task" -v

# 每日自动部署配置文件位置
/var/spool/cron/root
```

### 12. nagios

```
# 配置nagios server
vim /etc/sysconfig/send_nagios.cfg
```

### 13. airflow

```
# 手动触发任务
# 查看所有的任务
bash /opt/lzy-airflow/bin/lzy_airflow.sh list_dags
# 触发某个任务的执行
bash /opt/lzy-airflow/bin/lzy_airflow.sh trigger_dag [task-name]

# 查看帮助
bash /opt/lzy-airflow/bin/lzy_airflow.sh help

# 默认端口号
15002

# 删除失败任务
mysql -u airflow -p

use airflow;
show tables;
delete from task_instance where state!='success';
```

### 14. git

```
# 使用bundle文件 
# 修改 .git/config 把远程的位置修改为本地 bundle文件的地址，如下，再git pull即可，解决冲突后再把文件改回去，push到git 服务器上

[remote "origin"]
	url = D:/test/dolphin/dolphin.bundle
	fetch = +refs/heads/*:refs/remotes/origin/*
	puttykeyfile = ""
	
# 找回已删除代码
- 根据日志找到要恢复代码的版本号（sha）
- 使用git export version

# 如何把 rb 的代码合到 master
在 master 代码上右键，点击 merge ，选择要合并到 master 的分支，然后 merge ，可能需要解决冲突。
```