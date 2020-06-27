# 1. 简介

Ansible 是新出现的自动化运维工具，基于 **Python** 开发，集合了众多运维工具（puppet、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。
Ansible 是基于 paramiko 开发的,并且基于**模块化**工作，本身没有批量部署的能力。真正具有批量部署的是 ansible 所运行的模块，ansible 只是提供一种框架。ansible 不需要在远程主机上安装 client/agents，因为它们是基于 **SSH** 来和远程主机通讯的。ansible 目前已经已经被红帽官方收购，是自动化运维工具中大家认可度最高的，并且上手容易，学习简单。

# 2. 安装

建议安装 2.3 版本的 ansible，比较稳定，功能丰富，完全满足日常使用需求。

安装比较简单，使用 yum 即可，命令如下：

```shell
yum install ansible-2.3* -y
```

验证：

```shell
ansible --version
```

# 3. 配置

1. 生成拷贝 ssh key

   ```shell
   ssh-keygen -t rsa
   
   # 把公钥复制到所有的机器
   ssh-copy-id -i ~/.ssh/id_rsa.pub  <ip>
   ```

2. 修改配置加快速度

   ```shell
   vim /etc/ansible/ansible.cfg
   
   # 去掉主机名检查
   host_key_checking = False
   
   # 加快 ssh 的速度，强制用 ssh，防止自动选择用 python 的
   transport=ssh
   ssh_args=
   ```

3. 配置主机名

   这里配置后面需要操作的机器主机名。

   ```shell
   vim /etc/ansible/hosts
   
   [web]
   web01.wst.com
   [es]
   es01.wst.com
   es02.wst.com
   ```

   验证：

   ```shell
   ansible all --list-hosts
   ```

   如果可以正常列出所有的主机名，且不报错，即为正确。

# 4. 使用

执行 ssh agent ：

```shell
ssh-agent bash
ssh-add ~/.ssh/id_rsa
```

## 4.1 ping 一下所有机器，用于检测配置和网络

执行：

```shell
ansible all -m ping
```

输出：

```json
web01.wst.com | success >> {
    "changed": false,
    "ping": "pong"
}

es01.wst.com | success >> {
    "changed": false,
    "ping": "pong"
}

es02.wst.com | success >> {
    "changed": false,
    "ping": "pong"
}
```

## 4.2 创建文件夹

```shell
ansible all -a "mkdir /home/install"
```

## 4.3 拷贝文件到多台机器

这里是从本机拷贝文件到多台机器：

```shell
ansible all -m copy -a "src=/home/install/wst-2.3.1.tar.gz dest=/home/install"
```

输出如下：

```json
web01.wst.com | success >> {
    "changed": false,
    "checksum": "41fd5a16c4743ad6f4d4a662c477d39cd765ab55",
    "dest": "/home/install/wst-2.3.1.tar.gz",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/home/install/wst-2.3.1.tar.gz",
    "size": 188856400,
    "state": "file",
    "uid": 0
}
```

## 4.4 从目标机器拉取文件

```shell
ansible all -m fetch -a "src=/home/wst/install.log dest=logs"
```

输出如下：

```json
web01.wst.com | success >> {
    "changed": true,
    "checksum": "b304c6097fa7668cc051f2e650642f726ce4f6da",
    "dest": "/home/logs/install.log",
    "md5sum": "550d7136624952aaa42bdcfc9eaaf04d",
    "remote_checksum": "a304c6097fa7668cc051f2e650642f726ce4f6da",
    "remote_md5sum": null
}
```

## 4.5 批量执行 shell 命令

```shell
ansible all -m shell -a "df -h"
```

# 5. 剧本

**Playbook**（剧本），用于编排执行一些复杂的任务，下面先来看一个配置文件：

```yml
# web 机器的安装
- hosts: web_servers
  remote_user: root
  any_errors_fatal: true    
  max_fail_percentage: 0
  environment:
     LD_LIBRARY_PATH: ""
  vars_files:
     - "vars/install_version.yml"
  roles:
    - wst_web
  tags:
    - wst_web

# es 机器的安装
- hosts: es_servers
  remote_user: root
  any_errors_fatal: true   
  max_fail_percentage: 0 
  environment:
     LD_LIBRARY_PATH: ""
  vars_files:
     - "vars/install_version.yml"
  roles:
    - wst_es
  tags:
    - wst_es
```







