# 1.现象

在 centos7.2上使用 docker-19.03.6创建容器成功，但是容器无法启动，具体报错如下：

```
docker: Error response from daemon: driver failed programming external 
connectivity on endpoint:iptables failed:iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 3306 -j DNAT --to-destination 172.17.0.2:80 ! -i docker0 iptables: No chain/target/match by that name. (exit status 1)).
```

# 2.分析

报错很清晰：**执行 iptables 命令时出错了**

iptables的功能类似防火墙，用于管理数据包的传输，在 centos7的某个版本后已经不再单独存在，firewalld服务代替了 iptables。另外，只要 firewalld服务正常运行，iptables功能和命令可以正常使用，从现象来看二者有包含关系。

网上大部分资料都提到是因为 docker往 iptables里写入 nat规则失败，解决方法是**重启 docker服务**即可，在 centos7上可以执行如下命令：

```shell
systemctl restart docker
```

此方法在笔者的环境**行不通**，多次重启 docker服务后，容器依旧无法启动。

于是，换个思路，是否可以**禁止 docker往 iptables里写入 nat规则**？

——答案是肯定的

# 3.解决

查看 /etc/docker/daemon.json是否存在，不存在则创建，文件内容如下：

```json
{
	"iptables":false
}
```

然后重启 docker服务：

```sh
systemctl restart docker
```

至此，容器已经可以正常启动并对外提供服务。

目前，尚未发现关闭 iptables的副作用。

