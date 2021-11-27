# 1. 前言

虽然互联网上已经有了多个公开的可以直接使用的 yum源，但是针对于内网（局域网）这种无法连接互联网，又需要安装维护多台服务器的场景，就需要搭建一个内网的共享 yum源了。此时，就会涉及如何搭建本地 yum源、从公开 yum源同步 rpm包，以及手动添加 rpm包等操作，这就是本文的主要内容。

本文基于 CentOS7.9编写，应用场景为**局域网**多台服务器。

# 2. 搭建多主机共享的 YUM源

## 2.1 创建本地 YUM源

这里使用 CentOS操作系统镜像文件作为本地 yum源的基础 rpm包。

1. 创建一个目录用于挂载 ISO文件

   ```shell
   mkdir -p /mnt/iso/centos/7

2. 挂载 ISO文件

   ```shell
   mount -o loop CentOS-7-x86_64-DVD-2009.iso /mnt/iso/centos/7
   mkdir yum-repo/centos/7.9.2009/os/x86_64/ -p
   # 把 rpm包复制到建好的仓库目录中
   cp -r /mnt/iso/centos/7/* yum-repo/centos/7.9.2009/os/x86_64/
   ```

3. 创建 repo文件

   ```shell
   # 备份原有的 repo文件
   mv /etc/yum.repos.d /etc/yum.repos.d.bkp
   
   # 创建新的 repo文件
   mkdir /etc/yum.repos.d
   cd /etc/yum.repos.d
   
   vim local.repo
   # 内容如下
   [centos7]
   name=centos7
   baseurl=http://daphnis.centos7/centos/7.9.2009/os/$basearch/
   enable=1
   gpgcheck=0
   
   [update]
   name=update
   baseurl=http://daphnis.centos7/centos/7.9.2009/update/$basearch/
   enable=1
   gpgcheck=0
   ```

   其中 update这个仓库是给后面放软件的更新包用的，daphnis.centos7是主机名，需要**根据实际情况修改**

4. 验证

   ```shell
   yum clean all
   yum repolist
   ```

   能够看到刚刚创建的两个仓库即为正确，如下;

   ```
   Loaded plugins: fastestmirror
   Loading mirror speeds from cached hostfile
   centos7                                                                               | 3.6 kB  00:00:00
   update                                                                                | 2.9 kB  00:00:00

## 2.2 使用 Httpd共享 YUM源

1. 安装并启动 httpd

   ```shell
   yum install httpd -y
   systemctl start httpd
   
   # 设置开机启动
   systemctl enable httpd

2. 设置防火墙放开 80端口

   ```shell
   firewall-cmd --add-service=http --permanent
   firewall-cmd --add-port=80/tcp --permanent
   firewall-cmd --reload
   ```

3. 把 YUM仓库实际地址软链到 httpd的目录

   ```shell
   ln -s /opt/yum-repo/centos /var/www/html/
   ```

4. 验证

   使用另外一台服务器把 yum仓库的地址配成该服务器，修改 repo文件参考 2.1，然后能够正常使用 yum安转软件即为正确。

# 3. YUM源的维护

## 3.1 从公开 YUM源同步 RPM包

这里以同步清华 yum源中的两个目录为例。

1. 创建 repo文件

   ```shell
   vim /etc/yum.repos.d/ts.repo
   # 内容如下
   [ts-update]
   name=ts-update
   baseurl=https://mirrors4.tuna.tsinghua.edu.cn/centos/7.9.2009/updates/x86_64/
   enable=1
   gpgcheck=0
   
   [ts-kernel]
   name=ts-kernel
   baseurl=https://mirrors4.tuna.tsinghua.edu.cn/centos-altarch/7.9.2009/kernel/x86_64/
   enable=1
   gpgcheck=0
   ```

​		其中一个配置项代表需要同步的一个目录。

2. 安装依赖工具

   ```shell
   yum install yum-utils -y

3. 下载 RPM包到本地

   ```shell
   yum clean all
   cd /opt
   reposync -r ts-update -p ./ts-update/
   reposync -r ts-kernel -p ./ts-kernel/
   ```

4. 验证

   查看 /opt/ts-update和 /opt/ts-kernel这两个目录下是否有相应的 rpm文件

## 3.2 手动添加 RPM包

1. 准备好需要添加到仓库中的 rpm包

2. 把 rpm包上传到 /opt/yum-repo/centos/7.9.2009/update/x86_64/Packages

3. 安装依赖工具

   ```shell
   yum install createrepo -y
   ```

4. 更新 YUM仓库

   ```shell
   createrepo /opt/yum-repo/centos/7.9.2009/update/x86_64
   ```

 5. 验证

    ```shell
    # 方式1，查看包是否存在
    yum list |grep <pkg-name>
    
    # 方式2，如果是已安装软件的新版本，可以直接执行更新命令
    yum update <pkg-name>
    ```

# 4. 公开 YUM源

1. [官方CentOS基础包](http://mirror.centos.org/centos/7.9.2009)
2. [官方CentOS内核升级库](http://mirror.centos.org/altarch/7/kernel)
3. [清华开源镜像站](https://mirrors4.tuna.tsinghua.edu.cn/centos/7.9.2009/)
4. [阿里开源镜像站](http://mirrors.aliyun.com/centos/7.9.2009/)

备注：清华和阿里的镜像站是从 CentOS官网同步过来的，比较可靠且速度更快

# 5. 总结

本文基于笔者实际项目经验出发，以应用为主介绍了 yum源的搭建和维护，比较基础，后面根据需要可以进一步研究。
