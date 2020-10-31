# 前言

本文所有命令都基于 linux 发行版：**centos** ，centos 是目前工业上比较流行的服务器操作系统。



# 1. 查看系统信息

```shell
# centos 查看系统版本
cat /etc/redhat-release

# 查看主机名
hostname

# 防火墙
# 检查selinux
/usr/sbin/sestatus -v

# 查看系统时间
date

# 查看系统的开机信息 
dmesg
```



# 2. 目录切换

```shell
# 进到某个目录
cd <path>
# eg. cd /opt/test

# 回到上级目录
cd ..

# 回到上一次的目录
cd -

# 去到根目录
cd /
```



# 3. 查看文件列表

```shell
# 直接查看文件夹下的文件情况
ls

# 以列表方式查看文件情况
ls -l
# 一般创建别名 ll 用于代替

# 按时间降序排列显示当前文件夹下的文件
ll -t

# 统计当前目录下（含子目录）文件个数
ls -lR |grep "^-"|wc -l
```



# 4. 修改文件权限

linux 上经常会碰到文件存在权限问题，比如 没有**执行权限**、没有**写权限**。下面是一个文件的各种信息：

​	-rwxr-xr--. 1 root root 0 Dec  8 04:02 test.log

​	解释：- 表示这是一个文件，每一个文件都拥有三组，**rwx** 表示文件拥有者的权限，**r-x** 表示文件所属组的权限，**r--** 表示其他账户的权限

```shell
# r:4,w:2,x:1 rwx=4+2+1=7
# 修改文件权限为：文件拥有者和所属组有全部权限，而其他账户有读取权限
chmod -R 774 <file>
```

**注意：**网上好些解决文件权限问题都是将文件赋 **777**(所有账户都拥有该文件的读写执行权限) 的权限，这是不太可取的，会存在安全问题。



# 5. 安装和管理软件

linux 平台安装软件一般使用 **rpm** 和 **yum** ，其中在线安装软件推荐使用 **yum** 。



## 5.1 yum 相关操作

```shell
# 安装软件
yum -y install <soft-name>
# eg. yum -y install mysql

# 卸载软件
yum remove <soft-name>

# 查看yum源
yum repolist

# 查看可用的软件包
yum list |grep <soft-name>

# 查看已安装的软件包
yum list installed
# 查看是否已安装某个软件包
yum list installed |grep <soft-name>
```



## 5.2 rpm 相关操作

```shell
# 安装一个包 
rpm -ivh <soft-name>

# 卸载一个包
rpm -e <soft-name>

# 升级一个包 
rpm -Uvh <soft-name>

# 查找是否已安装某个 rpm包
rpm -qa |grep <soft-name>

# 列出一个未被安装进系统的RPM包文件中包含有哪些文件？ 
rpm -qilp <soft-name>

# 解压RPM包
rpm2cpio <soft-name> | cpio -div
```



# 6. 操作文本文件

这个操作是绝对的高频操作，诸如 **查看程序日志**、**编辑配置文件**、**查看文本文件**。而 **vim** 更是大多数开发、测试、运维人员的首选工具，注意 vim 不是 linux 必带工具，有的只带有 vi 工具，此时可以自行安装 vim 。



## 6.1 vim 相关命令

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



## 6.2 快速处理文本文件

光有 **vim** 可能还不够，**如何合并两个文本文件**？**如何对文件内容去重**？可以参考下面的命令：

```shell
# 批量合并文本文件
cat <dir>/*.txt > <filename>
# eg. cat article/*.txt > all_articles.txt ——将 article 目录下的所有 txt 文件合并为一个 txt 文件

# 合并两个文本文件，重复的行只保留一份
cat file1 file2 |sort |uniq > <file>

# 替换文本文件中某些字符 eg.把'\'替换成'\t'
sed -i 's/\\/\t/g' <file>

# 删除特定文本所在行
sed -i '/<text>/d' <file>
# eg. sed -i '/hello/d' test.txt ——删除内容含有 hello 的行

# 在文件中查找文本
# * : 表示当前目录所有文件，也可以是某个文件名
# -r 是递归查找
# -n 是显示行号
# -R 查找所有文件包含子目录
# -i 忽略大小写
grep -rn "hello,world!" *

# 查看文本文件行数
wc -l <file>

# 分割文件，下面的命令是把 test.log 文件每两行分割成一个子文件，子文件名以 sub-test 开头
# 一般的使用场景是把大文件切成一个个的小文件，便于之后处理
split -2 test.log sub-test

# 查看文件的前几行
head -n <num> <file>
# eg. head -n 2 test.txt ——查看 test.txt 的前两行

# 查看文件的最后几行
tail -n <num> <file>
# eg. tail -n 2 test.txt ——查看 test.txt 的最后两行

# 实时在屏幕显示文件增加的内容，这个常见用法是在屏幕实时打印程序运行日志
tailf <file>
# eg. tailf test.log
```



# 7. 查找文件和文件内容

这两个操作是绝对的高频操作。

```shell
# 查找目录下指定的文件
find <dir> -name <filename>
# 该命令还支持通配符
find /opt -name *redis*

# 查找文件中含有某个关键词的行
grep <keyword> test.txt

# grep
# 完全匹配
grep -w

# 查找含有 关键词1或关键词2的行
egrep '(<keyword1>|keyword2)' <file>
# eg. egrep '(kafka|redis)' test.log ——查找 test.log 中含有 kafka 或者 redis 的行
```

**grep** 的使用场景：比如 程序日志太多，不想一行一行的翻，直接查找文件中含有指定关键词的行：**grep ERROR logs/test.log **

**grep** 另外一个强大之处在于 可以**过滤任何屏幕显示的内容**，比如查找 kafka 相关的进程：**pe -ef |grep kafka**



# 8. 后台进程和服务管理

## 8.1 后台进程

```shell
# 查找含有某个关键字的进程
ps -ef |grep <keyword>
# eg. (ps -ef |grep kafka) ——将会查找 kafka 相关的进程

# 给进程ID 为 pid 的进程发送关闭信号，该进程主动关闭自己 ——推荐
kill <pid>

# 强制关闭 pid 进程，该进程不会收到任何消息 ——当进程出现问题时可以使用该命令
kill -9 <pid>

# 强制关闭进程名为 pname 的一系列进程 ——用于批量关闭进程
pkill -9 <pname>
# 也可以用 (killall -9 <pname>)

# 杀掉含有关键字 text 的进程
ps -ef|grep [<text>]|grep -v grep|awk  '{print "kill -9 " $2}' |sh
# eg. (ps -ef|grep python3|grep -v grep|awk  '{print "kill -9" $2}' |sh) ——强制杀死所有使用 python3命令 启动的进程
```



## 8.2 后台服务

后台服务大概可以理解为一种特殊的后台进程吧

```shell
# 查看服务状态
systemctl status <service name>

# 启动/关闭/重启 服务
systemctl start(stop|restart) <service name>
```



# 9. 解压和压缩文件

Linux 平台上最常见的压缩格式当属 **tar.gz** 了，推荐将压缩和解压密令都背下来，提高工作效率。

```shell
# 解压 tar.gz
tar -zxvf <file>.tar.gz -C <dir>
# 压缩
tar -zcvf <file>.tar.gz <dir> 

# 解压 tar
tar xvf <file>.tar -C <dir>
# 打包
tar -cvf <file>.tar <dir>

# 解压 zip
unzip <file>.zip
# 压缩
zip -r <file>.zip <dir>
```



# 10. 查看命令的位置和帮助信息

```shell
# 查看帮助信息
man <cmd>
info <cmd>

# 查看命令的位置
which <cmd>
```



# 11. 查看网络情况

```shell
# 查看网络信息
netstat -a

# 查看所有端口的使用情况
netstat -tunlp

# 查看某个端口占用情况
lsof -i:22
```



# 12. 远程复制文件

```shell
# 远程复制文件
# -r 递归复制
#           源路径  目标路径
scp -r <user>@ip:<spath> <dpath>
# eg. scp -r root@192.168.115.21:/opt/test-data /opt
```



# 13. 磁盘维护

虽然说研发的职责是开发程序，但是基本的服务器运维还是要了解一下的。

当在服务器上执行命令出现**明显卡顿**，或者服务器上某个**应用无法启动**时，可以尝试检查磁盘状态。

曾经出现过新手把**根目录塞满**的情况，导致应用无法启动。

```shell
# 查看当前文件夹下各个子文件夹和文件的空间占用情况
du -sh *
# 查看指定目录空间使用情况
df -h <dir>

# 每一个文件占用一个 inode ,inode 记录了文件的权限属性等信息
# 查看 inode 使用情况
df -i
```

**注意：**如果根目录满了，不要盲目的重启服务器（否则可能导致服务器无法启动），建议先使用 **du -sh** 命令查看根目录下的空间使用情况，再配合 **rm -rf** 命令释放磁盘空间。



# 14. 命令别名

```shell
# 查看别名
alias

# 定义别名
alias cls='clear'

# 取消别名
unalias cls
```



# 15. 命令行打印不同颜色的消息

这个命令其实算不得高频，但是**非常推荐在写 shell 脚本**的时候引入。

当使用 shell 脚本实施某些操作时，如果所有打印出来的消息都是同一个颜色，未免有些枯燥。反之，操作成功使用绿色的消息，操作失败使用红色的消息，岂不是有趣多了。

```shell
# echo 输出不同颜色的消息
echo -e "\033[30m 黑色字 \033[0m"
echo -e "\033[31m 红色字 \033[0m" 
echo -e "\033[32m 绿色字 \033[0m" 
echo -e "\033[33m 黄色字 \033[0m" 
echo -e "\033[34m 蓝色字 \033[0m" 
echo -e "\033[35m 紫色字 \033[0m" 
echo -e "\033[36m 天蓝字 \033[0m" 
echo -e "\033[37m 白色字 \033[0m"
```



# 总结

这些命令和操作虽然不是很完善，但是如果全部掌握了，去各地现场**部署系统**、**排查问题**、**处理数据**等常见操作基本也不虚了..





