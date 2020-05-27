# 1. 前言

最近入坑 NLP 了，开始正儿八经玩 Python ，那么 pip 的使用就必须要掌握了。

# 2. 修改 Pip 仓库地址为国内源

这个跟 maven 一样如果使用默认的国外仓库，很多时候速度感人 ..如果想按时下班，那么更换国内源是必须的。

**国内常用的 Pip 地址仓库如下：**

- **阿里云**， http://mirrors.aliyun.com/pypi/simple/
- **中国科技大学**， https://pypi.mirrors.ustc.edu.cn/simple/
- **清华大学**， https://pypi.tuna.tsinghua.edu.cn/simple/
- **豆瓣**，http://pypi.douban.com/simple/

**使用国内的仓库源一般有两种方式：**

1. **临时使用**，直接在命令行指定使用的仓库地址即可

   ```shell
   pip -i https://pypi.mirrors.ustc.edu.cn/simple/ install numpy
   ```

   这个很好理解，**-i** 后面跟的就是本次安装使用的仓库地址。

2. **永久生效**，需要修改配置文件

   windows 平台在 **C:\Users\YourName\pip** 下创建 pip.ini 文件（pip 文件夹不存在手动创建即可），文件内容如下：

   ```properties
   [global]
   index-url = https://pypi.tuna.tsinghua.edu.cn/simple
   [install]
   trusted-host=mirrors.aliyun.com
   ```

   **trusted-host** 这个配置是指的信任这个主机名，否则在使用 pip 安装包时会告警，不受信任的主机地址。

# 3. 基础命令

虽然 **pip -h** 能很方便的看到所有的命令，但是记一下比较常用的命令还是可以的。

下面介绍几个比较实用的基础命令：

```shell
# 查看 pip 版本信息
pip -V

# 安装/卸载包
pip install/uninstall pyspark

# 查看可安装包的版本
pip install pyspark==

# 查看已安装的包
pip list

# 查看已安装包的信息，这里面比较有用的就是版本信息（吐槽下 Python 是经常因为包的版本不对导致一些奇怪的问题）
pip show pyspark

# 检查包的依赖项
pip check pyspark

# 以 requirements 的格式输出已安装的包
# 这个命令可以用于辅助下载已安装的包，然后到另一台机器离线安装
pip freeze
```

# 4. 批量下载安装 WHL 格式的包

**WHL**，其实就是把编译好的 Python 文件打包到了一起，有点类似 Java 的 jar 包，省去了编译过程，可以直接安装。

不过还是来看下官方的给的解释：

> Python Wheels are the new standard of Python distribution and are intended toreplace eggs.

这个适合在无法联网的机器上快速构建 Python 的依赖环境。

**注意：**下载 WHL 包的机器和后面需要离线安装包的机器，Python 版本和机器位数要一致，否则后面安装时会报错。

## 4.1 批量下载 WHL 包

1. 准备好 requirements 文件

   例如：requirements.txt 内容如下，里面放需要下载的包名称和版本

   ```
   py4j==0.10.7
   pylint==1.9.3
   pyspark==2.3.0
   ```

2. 使用下载命令

   ```shell
   pip download -d spark-libs -r requirements.txt
   ```

   **-d** 后面跟的是下载的包存放目录

## 4.2 批量安装 WHL 包

比较简单，需要使用下载时用到的 requirements.txt，命令如下：

```shell
pip install --no-index --find-links=d:\spark-libs -r requirements.txt
```

**--find-links** 后面跟的是存放 whl 包的路径。

**补充**：如果安装失败，一般是包对应的 Python 版本或者位数不对，需要去下载正确版本和位数的包，然后进行替换即可。比如提示 py4j 有问题，可以打开下面的地址手动下载正确的包：

https://pypi.tuna.tsinghua.edu.cn/simple/py4j

# 5. 总结

Pip 用来管理 Python 包还是很方便的，暂时就写这么多，后面发现好的技巧再来补充下。

