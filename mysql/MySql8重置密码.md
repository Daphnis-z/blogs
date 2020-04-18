# MySql8 重置密码踩坑记

**为什么要重置密码呢？**

MySql8 首次安装完成后会为 root 生成一个随机密码，可以在日志（/var/log/mysqld.log）里面找到。手动安装都不会有问题，修改密码参考网上的教程很简单的。但是如果是自动部署呢？去日志里面捞这个密码可不容易。

## 先贴一下官网重置密码的步骤，加入一些自己的经验：

下面针对的是 Linux 平台重置密码的步骤，**其他平台可以参考文末官网链接**。

### 1. 停止 MySql 服务

下面两种方式都可以：

```shell
# 1.官网给的命令，其实就是拿到 mysqld 的进程ID ，然后 kill 一下
kill `cat /var/run/mysqld/mysqld.pid`

# 2.使用 service 命令
# centos6
service mysqld stop
# centos7
systemctl stop mysqld
```

### 2. 创建文本文件，并在里面加上修改 root 密码的 sql

比如创建一个 edit-root-pwd.sql 

```shell
vim edit-root-pwd.sql

# 加入下面的sql（默认情况下，mysql 密码规则是 大小写字母、数字、特殊符号都要有）
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewRootPwd@123';
```

### 3. 启动 MySql 并指定初始化文件

直接执行命令：

```shell
mysqld --init-file=/tmp/edit-root-pwd.sql &
```

到这里一切都十分顺利，然而，mysql 竟然无法启动了 ！

看下 mysql 启动日志，报有文件无法读取，没有权限。马上去看 mysql 数据目录（/var/lib/mysql/），里面竟然有好几个文件属于 root 。百度无果后，重新回到官网：

> The instructions assume that you will start the MySQL server from the Unix login account that you normally use for running it. For example, if you run the server using the `mysql` login account, you should log in as `mysql` before using the instructions. Alternatively, you can log in as `root`, but in this case you *must* start [**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)with the [`--user=mysql`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_user) option. If you start the server as `root` without using [`--user=mysql`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_user), the server may create `root`-owned files in the data directory, such as log files, and these may cause permission-related problems for future server startups. If that happens, you will need to either change the ownership of the files to `mysql`or remove them. 

**Alternatively, you can log in as `root`, but in this case you must start mysqld with the --user=mysql option.**

只怪自己不仔细，官网已经提示了，要么使用 mysql 用户登录系统，要么使用 root 用户登录时要加一下参数：

**--user=mysql**

修改下 mysql 启动命令为：

```shell
mysqld --user=mysql --init-file=/tmp/edit-root-pwd.sql &
```

至此密码修改完成，完美的解决了问题。



## 总结：

看官方文档时一定要仔细，怀疑官方文档前，先审视下自己的操作，看下是否漏了什么步骤，有时候踩的坑是自己挖的。

贴一下官网文档地址：[B.4.3.2 How to Reset the Root Password](https://dev.mysql.com/doc/refman/8.0/en/resetting-permissions.html)