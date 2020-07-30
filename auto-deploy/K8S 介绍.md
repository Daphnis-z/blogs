# 1. 简介

K8S，全称是 **Kubernetes** 。一个开源的，用于管理云平台中多个主机上的容器化的应用，目标是让部署容器化的应用简单并且高效，提供了一种应用部署，规划，更新，维护的机制。

Kubernetes 一个核心的特点就是能够自主的管理容器来保证云平台中的容器按照用户的期望状态运行着，负责创建、启动、重启容器，确保服务长期可用。

## 1.1 核心组件

1. **etcd** 保存了整个集群的状态；

2. **apiserver** 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
3. **controller manager** 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
4. **scheduler** 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上；
5. **kubelet** 负责维护容器的生命周期，同时也负责 Volume（CVI）和网络（CNI）的管理；
6. **Container runtime** 负责镜像管理以及Pod和容器的真正运行（CRI）；
7. **kube-proxy** 负责为 Service 提供 cluster 内部的服务发现和负载均衡；

## 1.2 分层架构

Kubernetes 设计理念和功能其实就是一个类似 Linux 的分层架构，如下：

- **核心层**

  Kubernetes 最核心的功能，对外提供 API 构建高层应用，对内提供插件式应用执行环境

- **应用层**

  部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS 解析等）

- **管理层**

  系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态 Provision 等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy 等）

- **接口层**

  kubectl 命令行工具、客户端 SDK 以及集群联邦

- **生态系统**

  在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴：

  1. Kubernetes 外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS 应用、ChatOps 等
  
  2. Kubernetes 内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等

## 1.3 名词解释

1. **Pod**

   Pod 是在 K8s 集群中运行部署应用或服务的**最小单元**，支持多容器。其设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

2. **RC**

   Replication Controller，副本控制器。RC 是 K8s 集群中最早的保证 Pod 高可用的 API 对象。通过监控运行中的 Pod 来保证集群中运行指定数目的 Pod 副本。指定的数目可以是多个也可以是 1个；少于指定数目，RC 就会启动运行新的 Pod 副本；多于指定数目，RC 就会杀死多余的 Pod 副本。

3. **RS**

   Replica Set，副本集。RS 是新一代 RC，提供同样的高可用能力，区别主要在于 RS 后来居上，能支持更多种类的匹配模式。副本集对象一般不单独使用，而是作为 Deployment 的理想状态参数使用。

4. **Deployment**

   部署，表示用户对 K8s 集群的一次更新操作，是一个比 RS 应用模式更广的 API 对象，可以是创建一个新的服务，更新一个服务，也可以是滚动升级一个服务。滚动升级一个服务，实际是创建一个新的 RS，然后逐渐将新 RS 中副本数增加到理想状态，将旧 RS 中的副本数减小到 0的复合操作。

5. **Service**

   在 K8s 集群中，客户端需要访问的服务就是 Service 对象。每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部通过虚拟 IP 访问一个服务。在 K8s 集群中微服务的负载均衡是由 Kube-proxy 实现的。

6. **Job**

   任务，K8s 用来控制批处理型任务的 API 对象。批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job 管理的 Pod 根据用户的设置把任务成功完成就自动退出了。

7. **DaemonSet**

   后台支撑服务集，长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的 Pod，有些节点上又没有这类 Pod运行；而后台支撑型服务的核心关注点在 K8s 集群中的节点（物理机或虚拟机），要保证每个节点上都有一个此类 Pod 运行。节点可能是所有集群节点也可能是通过 nodeSelector 选定的一些特定节点。典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持 K8s 集群运行的服务。

8. **PetSet**

   有状态服务集。在云原生应用的体系里，有下面两组近义词；第一组是无状态（stateless）、牲畜（cattle）、无名（nameless）、可丢弃（disposable）；第二组是有状态（stateful）、宠物（pet）、有名（having name）、不可丢弃（non-disposable）。RC 和 RS 主要是控制提供无状态服务的，其所控制的Pod的名字是随机设置的，一个Pod出故障了就被丢弃掉，在另一个地方重启一个新的Pod，名字变了、名字和启动在哪儿都不重要，重要的只是 Pod 总数；而 PetSet 是用来控制有状态服务，PetSet 中的每个 Pod 的名字都是事先确定的，不能更改。

9. **Federation**

   集群联邦。在云计算环境中，服务的作用距离范围从近到远一般可以有：同主机（Host，Node）、跨主机同可用区（Available Zone）、跨可用区同地区（Region）、跨地区同服务商（Cloud Service Provider）、跨云平台。K8s 的设计定位是单一集群在同一个地域内，因为同一个地区的网络性能才能满足 K8s 的调度和计算存储连接要求。而联合集群服务就是为提供跨 Region 跨服务商K8s集群服务而设计的。

10. **Volume**

    存储卷，生命周期和作用范围是一个 Pod。每个 Pod 中声明的存储卷由 Pod 中的所有容器共享。K8s 支持非常多的存储卷类型，特别的，支持多种公有云平台的存储，包括 AWS，Google 和 Azure 云；支持多种分布式存储包括 GlusterFS 和 Ceph；也支持较容易使用的主机本地目录 hostPath 和 NFS。

11. **PV 和 PVC**

    Persistent Volume（持久存储卷）和 Persistent Volume Claim（持久存储卷声明）。PV 和 PVC 使得 K8s 集群具备了存储的逻辑抽象能力，使得在配置 Pod 的逻辑里可以忽略对实际后台存储技术的配置，而把这项配置的工作交给 PV 的配置者，即集群的管理者。

12. **Node**

    节点，K8s 集群中的计算能力由 Node 提供，最初 Node 称为服务节点 Minion，后来改名为 Node。K8s 集群中的 Node 也就等同于Mesos 集群中的 Slave 节点，是所有 Pod 运行所在的工作主机，可以是物理机也可以是虚拟机。

13. **Secret**

    密钥对象，Secret 是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象。使用 Secret 的好处是可以避免把敏感信息明文写在配置文件里。

14. **UA 和 SA**

    User Account（用户账户）和 Service Account（服务账户）。用户帐户为人提供账户标识，而服务账户为计算机进程和 K8s 集群中运行的 Pod 提供账户标识。用户帐户和服务帐户的一个区别是作用范围；用户帐户对应的是人的身份，人的身份与服务的 namespace 无关，所以用户账户是跨 namespace 的；而服务帐户对应的是一个运行中程序的身份，与特定 namespace 是相关的。

15. **Namespace**

    名字空间为 K8s 集群提供虚拟的隔离作用，K8s 集群初始有两个名字空间，分别是默认名字空间 default 和系统名字空间 kube-system，除此以外，管理员可以可以创建新的名字空间满足需要。

16. **RBAC 访问授权**

    Role-based Access Control（基于角色的访问控制），在RBAC中，访问策略可以跟某个角色关联，具体的用户在跟一个或多个角色相关联。

# 2. 安装

Kubernetes 可以在多种平台运行，从笔记本电脑，到云服务商的虚拟机，再到机架上的裸机服务器。要创建一个 Kubernetes 集群，根据不同场景需要做的也不尽相同，可能是运行一条命令，也可能是配置自己的定制集群。

## 2.1 解决方案

### 2.1.1 本地服务器方案

本地服务器方案再一台物理机上创建拥有一个或者多个 Kubernetes 节点的单机集群。创建过程是全自动的，且不需要任何云服务商的账户。但是这种单机集群的规模和可用性都受限于单台机器。

本地服务器方案有：

- 本地 Docker
- Vagrant (任何支持 Vagrant 的平台：Linux，MacOS，或者 Windows。)
- 无虚拟机本地集群 (Linux)

### 2.1.2 托管方案

Google Container Engine 提供创建好的Kubernetes集群。

### 2.1.3 全套云端方案

以下方案让你可以通过几个命令就在很多 IaaS 云服务中创建Kubernetes集群，并且有很活跃的社区支持。

- GCE
- AWS
- Azure

### 2.1.4 定制方案

Kubernetes 可以在云服务提供商和裸机环境运行，并支持很多基本操作系统。

### 2.1.5 云

以下是一些云服务商或云操作系统支持的方案。

- AWS + coreos
- GCE + CoreOS
- AWS + Ubuntu
- Joyent + Ubuntu
- Rackspace + CoreOS

### 2.1.6 私有虚拟机

- Vagrant（采用CoreOS和flannel）
- CloudStack（采用Ansible，CoreOS和flannel）
- Vmware（采用Debian）
- juju.md（采用Juju，Ubuntu和flannel）
- Vmware（采用CoreOS和flannel）
- libvirt-coreos.md（采用CoreO）
- oVirt
- libvirt（采用Fedora和flannel）
- KVM（采用Fedora和flannel）

### 2.1.7 裸机

- Offline（无需互联网，采用CoreOS和flannel）
- fedora/fedora_ansible_config.md
- Fedora单节点
- Fedora多节点
- Centos
- Ubuntu
- Docker多节点

### 2.1.8 集成

Kubernetes on Mesos（采用GCE）

## 2.2 基于 Docker 本地运行 k8s

这里以 Linux 系统为例。

### 2.2.1 准备

1. 在机器上安装好 docker

2. 内核必须支持 memory and swap accounting，确认 linux 开启了如下配置：

   ```
   CONFIG_RESOURCE_COUNTERS=y
   CONFIG_MEMCG=y
   CONFIG_MEMCG_SWAP=y
   CONFIG_MEMCG_SWAP_ENABLED=y
   CONFIG_MEMCG_KMEM=y
   ```

3. 以命令行参数方式，在内核启动时开启 memory and swap accounting 选项：

   ```
   GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
   ```

   传给内核：

   ```
   $cat /proc/cmdline
   BOOT_IMAGE=/boot/vmlinuz-3.18.4-aufs root=/dev/sda5 ro cgroup_enable=memory
   swapaccount=1
   ```

### 2.2.2 启动相应服务

1. **运行 Etcd**

   ```shell
   docker run --net=host -d gcr.io/google_containers/etcd:2.0.12 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data
   ```

2. **启动 master**

   ```shell
   docker run \
       --volume=/:/rootfs:ro \
       --volume=/sys:/sys:ro \
       --volume=/dev:/dev \
       --volume=/var/lib/docker/:/var/lib/docker:ro \
       --volume=/var/lib/kubelet/:/var/lib/kubelet:rw \
       --volume=/var/run:/var/run:rw \
       --net=host \
       --pid=host \
       --privileged=true \
       -d \
       gcr.io/google_containers/hyperkube:v1.0.1 \
       /hyperkube kubelet --containerized --hostname-override=&quot;127.0.0.1&quot; --address=&quot;0.0.0.0&quot; --api-servers=http://localhost:8080 --config=/etc/kubernetes/manifests
   ```

3. **运行 service proxy**

   ```shell
   docker run -d --net=host --privileged gcr.io/google_containers/hyperkube:v1.0.1 /hyperkube proxy --master=http://127.0.0.1:8080 --v=2
   ```

### 2.2.3 验证

**下载 kubectl 二进制程序进行测试：**

```shell
kubectl get nodes
```

输出：

```
NAME LABELS STATUS
127.0.0.1 Ready
```

**运行一个应用：**

```
kubectl -s http://localhost:8080 run nginx --image=nginx --port=80
```

运行 docker ps 应该就能看到 nginx 在运行。

**暴露为 service：**

```shell
kubectl expose rc nginx --port=80
```

运行以下命令来获取刚才创建的 service 的 IP 地址。有两个 IP，第一个是内部的（CLUSTER_IP），第二个是外部的负载均衡IP。

```
kubectl get svc nginx
```

同样也可以通过运行以下命令只获取第一个IP（CLUSTER_IP）：

```
kubectl get svc nginx --template={{.spec.clusterIP}}
```

通过第一个IP（CLUSTER_IP）访问服务：

```
curl <insert-cluster-ip-here>
```

# 3. 使用

这里以 spark 为例简单介绍下如何使用 k8s。

下面介绍如何使用 Kubernetes 和 Docker 创建一个能够使用的 Apache Spark 集群。这里使用 Spark 的单例模式创建一个Spark master 节点服务和一系列 Spark workers 节点。

## 3.1 启动 Master 服务

Master 服务是一个 Spark 集群的主服务（或头服务）。

使用 **[examples/spark/spark-master.json](http://kubernetes.io/v1.0/examples/spark/spark-master.json)** 文件创建运行在Master服务中的pod。

```shell
kubectl create -f examples/spark/spark-master.json
```

然后使用 **[examples/spark/spark-master-service.json](http://kubernetes.io/v1.0/examples/spark/spark-master-service.json)** 文件创建一个逻辑服务端点供Spark workers节点使用连接Matser pod。

```shell
kubectl create -f examples/spark/spark-master-service.json
```

**检查Master节点使用运行并且能够连接：**

```shell
kubectl get pods

# 输出如下
NAME                                           READY     STATUS    RESTARTS   AGE
[...]
spark-master                                   1/1       Running   0          25s
```

检查日志查看 master 节点的状态：

```shell
kubectl logs spark-master

# 输出如下
starting org.apache.spark.deploy.master.Master, logging to /opt/spark-1.4.0-bin-hadoop2.6/sbin/../logs/spark--org.apache.spark.deploy.master.Master-1-spark-master.out
Spark Command: /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java -cp /opt/spark-1.4.0-bin-hadoop2.6/sbin/../conf/:/opt/spark-1.4.0-bin-hadoop2.6/lib/spark-assembly-1.4.0-hadoop2.6.0.jar:/opt/spark-1.4.0-bin-hadoop2.6/lib/datanucleus-api-jdo-3.2.6.jar:/opt/spark-1.4.0-bin-hadoop2.6/lib/datanucleus-rdbms-3.2.9.jar:/opt/spark-1.4.0-bin-hadoop2.6/lib/datanucleus-core-3.2.10.jar -Xms512m -Xmx512m -XX:MaxPermSize=128m org.apache.spark.deploy.master.Master --ip spark-master --port 7077 --webui-port 8080
========================================
15/06/26 14:01:49 INFO Master: Registered signal handlers for [TERM, HUP, INT]
15/06/26 14:01:50 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
15/06/26 14:01:51 INFO SecurityManager: Changing view acls to: root
15/06/26 14:01:51 INFO SecurityManager: Changing modify acls to: root
15/06/26 14:01:51 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(root); users with modify permissions: Set(root)
15/06/26 14:01:51 INFO Slf4jLogger: Slf4jLogger started
15/06/26 14:01:51 INFO Remoting: Starting remoting
15/06/26 14:01:52 INFO Remoting: Remoting started; listening on addresses :[akka.tcp://sparkMaster@spark-master:7077]
15/06/26 14:01:52 INFO Utils: Successfully started service 'sparkMaster' on port 7077.
15/06/26 14:01:52 INFO Utils: Successfully started service on port 6066.
15/06/26 14:01:52 INFO StandaloneRestServer: Started REST server for submitting applications on port 6066
15/06/26 14:01:52 INFO Master: Starting Spark master at spark://spark-master:7077
15/06/26 14:01:52 INFO Master: Running Spark version 1.4.0
15/06/26 14:01:52 INFO Utils: Successfully started service 'MasterUI' on port 8080.
15/06/26 14:01:52 INFO MasterWebUI: Started MasterWebUI at http://10.244.2.34:8080
15/06/26 14:01:53 INFO Master: I have been elected leader! New state: ALIVE
```

## 3.2 启动 Spark workers

在 Spark 集群中 Spark workers 做繁重的工作。它们为你的程序提供计算资源和数据缓存的能力。

Spark workers 需要 Master 服务支持才能运行。

使用 **[examples/spark/spark-worker-controller.json](http://kubernetes.io/v1.0/examples/spark/spark-worker-controller.json)** 文件创建[复制控制器](https://www.kubernetes.org.cn/replication-controller-kubernetes)管理workers pod。

```shell
kubectl create -f examples/spark/spark-worker-controller.json
```

**检查 workers 节点是否运行：**

```shell
kubectl get pods

# 输出如下
NAME                                            READY     STATUS    RESTARTS   AGE
[...]
spark-master                                    1/1       Running   0          14m
spark-worker-controller-hifwi                   1/1       Running   0          33s
spark-worker-controller-u40r2                   1/1       Running   0          33s
spark-worker-controller-vpgyg                   1/1       Running   0          33s

kubectl logs spark-master

# 输出如下
[...]
15/06/26 14:15:43 INFO Master: Registering worker 10.244.2.35:46199 with 1 cores, 2.6 GB RAM
15/06/26 14:15:55 INFO Master: Registering worker 10.244.1.15:44839 with 1 cores, 2.6 GB RAM
15/06/26 14:15:55 INFO Master: Registering worker 10.244.0.19:60970 with 1 cores, 2.6 GB RAM
```

## 3.3 启动 Spark 客户端

**获取 Master 服务的地址和端口：**

```shell
kubectl get service spark-master

# 输出如下
NAME           LABELS              SELECTOR            IP(S)          PORT(S)
spark-master   name=spark-master   name=spark-master   10.0.204.187   7077/TCP
```

使用 SSH 连接集群中的一个节点，name 可以通过 kubectl get nodes 命令获得。

```shell
kubectl get nodes

# 输出如下
NAME                     LABELS                                          STATUS
kubernetes-minion-5jvu   kubernetes.io/hostname=kubernetes-minion-5jvu   Ready
kubernetes-minion-6fbi   kubernetes.io/hostname=kubernetes-minion-6fbi   Ready
kubernetes-minion-8y2v   kubernetes.io/hostname=kubernetes-minion-8y2v   Ready
kubernetes-minion-h0tr   kubernetes.io/hostname=kubernetes-minion-h0tr   Ready

gcloud compute ssh kubernetes-minion-5jvu --zone=us-central1-b

# 输出如下
Linux kubernetes-minion-5jvu 3.16.0-0.bpo.4-amd64 #1 SMP Debian 3.16.7-ckt9-3~deb8u1~bpo70+1 (2015-04-27) x86_64

=== GCE Kubernetes node setup complete ===
```

一旦登陆成功就可以使用 Spark 基础镜像了。在镜像中有一个脚本用来设置基于 Master 的 IP 和端口环境。

```shell
docker run -it gcr.io/google_containers/spark-base
 . /setup_client.sh 10.0.204.187 7077
pyspark

# 输出如下
Python 2.7.9 (default, Mar  1 2015, 12:57:24) 
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
15/06/26 14:25:28 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 1.4.0
      /_/
Using Python version 2.7.9 (default, Mar  1 2015 12:57:24)
SparkContext available as sc, HiveContext available as sqlContext.
>>> import socket
>>> sc.parallelize(range(1000)).map(lambda x:socket.gethostname()).distinct().collect()
['spark-worker-controller-u40r2', 'spark-worker-controller-hifwi', 'spark-worker-controller-vpgyg']
```

















