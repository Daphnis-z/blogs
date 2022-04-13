# 1.现象

Elasticsearch、Hadoop和 MongoDB等都需要修改 Linux里面**最大打开文件数**这个配置，即下面的 **open files**

```shell
[root@localhost opt]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 515003
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 515003
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

常规修改 open files只需要在 /etc/security/**limits.conf**文件里面添加如下配置：

```
*       hard    nofile   65536
*       soft    nofile   65536
```

笔者的环境出于安全性考虑将 openssh由 [**OpenSSH_7.4p1**, OpenSSL 1.0.2k-fips]升级到了 [**OpenSSH_8.8p1**, OpenSSL 1.1.1m]，这个时候问题就来了，明明已经将 open files修改为 65536了，但是无论是使用 ulimit -a还是 ulimit -n命令均**显示的默认值 1024**。

笔者服务器版本：CentOS Linux release 7.9.2009 (Core)

# 2.分析

SSH一般都是用于连接远程服务器或者远程服务器之间通讯的，为何为影响到 Linux系统的配置项呢？如果能影响，那么**决定 open files这个配置项的值的因素有哪些呢**？

网上的大部分文章都只提到了使用 ulimit命令或者修改 limits.conf文件来实现修改 open files配置，但是通过细心查看发现 PAM可能跟这个配置项也有关。

**PAM**，Pluggable Authentication Modules，可插拔认证模块。简而言之，一种高层次的统一安全认证机制。

再查阅资料后发现，openssh也是支持 PAM的。

# 3.解决

第一步，修改 openssh配置文件，开启对 PAM的支持：

```shell
vim /etc/ssh/sshd_config
在文件末尾添加：UsePAM yes
```

第二步，重启 ssd服务：

```shell
systemctl restart sshd
```

第三步，验证结果：

```shell
[root@localhost opt]# ulimit -n
65536
```

