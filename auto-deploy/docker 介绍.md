# 1. 简介

Docker 是一个开源的**应用容器引擎**，基于 Go 语言并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包应用以及依赖包到一个轻量级、可移植的镜像中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

## 1.1 应用场景

- Web 应用的自动化打包和发布
- 自动化测试和持续集成、发布
- 在服务型环境中部署和调整数据库或其他的后台应用
- 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境

## 1.2 优点

Docker 是一个用于开发，交付和运行应用程序的开放平台。可以隔离应用程序与基础架构，从而可以快速部署、测试和交付软件。

具体优势如下：

- **快速，一致地交付应用程序**

  Docker 允许开发人员使用提供的应用程序或服务的本地容器在标准化环境中工作，从而简化了开发的生命周期。容器非常适合持续集成和持续交付（CI / CD）工作流程，可以考虑以下示例方案：

  - 开发人员在本地编写代码，并使用 Docker 容器与同事共享他们的工作。
  - 使用 Docker 将其应用程序推送到测试环境中，并执行自动或手动测试。
  - 当开发人员发现错误时，可以在开发环境中对其进行修复，然后将其重新部署到测试环境中，以进行测试和验证。
  - 测试完成后，将修补程序推送给生产环境，就像将更新的镜像推送到生产环境一样简单。

- **响应式部署和扩展**

  Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。

  Docker 的可移植性和轻量级的特性，还可以减轻动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务。

- **在同一硬件上运行更多工作负载**

  Docker 轻巧快速，为基于虚拟机管理程序提供了可行、经济、高效的替代方案，因此可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，可以用更少的资源做更多的事情。

## 1.3 名词解释

- **镜像（Image）**

  Docker 镜像（Image），内部是一个精简的操作系统（OS），同时还包含应用运行所必须的文件和依赖包。开发人员把自己开发完成的应用程序及其依赖打包到一起，生成一个镜像，需要运行应用时只需要根据镜像生成应用实例即可，非常便捷。

- **容器（Container）**

  镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是**静态的定义**，容器是镜像**运行时的实体**。容器可以被创建、启动、停止、删除、暂停等。

- **仓库（Repository）**

  仓库可看成一个代码控制中心，用来保存镜像。

# 2. 安装

这里以常用的 centos（docker 默认支持的是 centos7 及以上版本） 为例介绍 docker 的安装。下面介绍 2种安装方式。

## 2.1 一键安装

由于使用官方提供的脚本安装 docker 速度较慢，一般推荐使用国内 daocloud 一键安装命令：

```shell
curl -sSL https://get.daocloud.io/docker | sh
```

## 2.2 手动安装

1. **设置仓库**

   由于官方的源速度太慢，同样也推荐使用国内的源，比如 阿里云或者清华大学：

   ```shell
   # 阿里云
   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   
   # 清华大学
   yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
   ```

2. **安装 Docker Engine-Community**

   一般安装最新版本的 Docker Engine-Community 和 containerd 即可，命令如下：

   ```shell
   yum install docker-ce -y
   ```

## 2.3 验证

Docker 安装完默认未启动。并且已经创建好 docker 用户组，但该用户组下没有用户。

先启动 docker：

```shell
systemctl start docker
```

运行 hello-world：

```shell
docker run hello-world
```

出现下面类似的信息说明 docker 安装完成且运行正常：

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

# 3. 使用

## 3.1 镜像

1. **查看本地镜像**

   ```shell
   docker images
   
   # 输出类似下面
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   redis               latest              235592615444        5 weeks ago         104MB
   hello-world         latest              bf756fb1ae65        6 months ago        13.3kB
   ubuntu              15.10               9b9cb95443b5        3 years ago         137MB
   ```

2. **下载镜像**

   以下载 13.10 版本的 ubuntu 为例：

   ```shell
   docker pull ubuntu:13.10
   ```

3. **查找镜像**

   以查找 redis 相关镜像为例：

   ```shell
   docker search httpd
   
   # 输出类似下面
   NAME              DESCRIPTION                            STARS    OFFICIAL  AUTOMATED
   arm32v7/redis        Redis is an open source key-value store that…    21      [OK]
   bitnami/redis-sentinel  Bitnami Docker Image for Redis Sentinel         14             [OK]
   webhippie/redis       Docker images for Redis                   12             [OK]
   redislabs/redisgraph   A graph database module for Redis             11             [OK]
   ...
   
   # OFFICIAL: 是否 docker 官方发布
   # STARS: 类似 Github 里面的 star，表示点赞、喜欢的意思。
   ```

4. **删除镜像**

   以删除 redis 为例：

   ```shell
   docker rmi hello-world
   ```

5. **创建镜像**

   当 docker 仓库中的镜像无法满足要求时，有下面两种方式可以对镜像进行修改：

   - **从已经创建的容器中更新镜像，并且提交这个镜像**

     更新镜像之前需要先使用镜像创建一个容器：

     ```shell
     docker run -t -i ubuntu:15.10 /bin/bash
     ```

     容器正常运行后，可以进行相应修改，然后提交：

     ```
     docker commit -m="update something" -a="daphnis" e218edb10165 daphnis/ubuntu:v5
     
     # -m 描述信息
     # -a 指定镜像作者
     # e218edb10165 容器 ID
     # daphnis/ubuntu:v5 指定要创建的目标镜像名称
     ```

     更新完成后可以使用 **docker images** 命令查看镜像是否提交成功。

   - **使用 Dockerfile 指令来创建一个新的镜像** ——这个后面单独进行介绍

## 3.2 容器

下载完镜像后可以使用下面的命令启动一个容器，这里以 ubuntu 为例：

```shell
docker run -it ubuntu /bin/bash
# -i 交互式操作
# -t 终端
# /bin/bash 放在镜像名称后面的命令
```

1. **查看容器**

   ```shell
   # 查看正在运行的容器
   docker ps
   
   # 查看所有的容器
   docker ps -a
   ```

2. **启动停止容器**

   ```shell
   # 根据容器ID 启动容器（查看容器ID 可以使用 docker ps -a 命令）
   docker start <container-id>
   
   # 根据容器ID 停止容器
   docker stop <container-id>
   
   # 后台运行一个容器
   docker run -itd --name ubuntu-test ubuntu /bin/bash
   # -d 指定后台运行
   ```

3. **进入容器**

   在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：

   - **docker attach**

     ```shell
     docker attach <container-id>
     ```

   - **docker exec**：推荐使用 docker exec 命令，因为用这个命令退出容器终端，不会导致容器的停止

     ```shell
     # 进入容器内部目录
     docker exec -it <container-id|container-name> /bin/bash
     # eg. docker exec -it redis101 /bin/bash
     ```

4. **导入和导出容器**

   ```shell
   # 导出容器
   docker export <container-id> > ubuntu.tar
   
   # 导入容器到 test/ubuntu 镜像
   cat docker/ubuntu.tar | docker import - test/ubuntu:v5
   
   # 指定目录或者 URL 来导入
   docker import http://example.com/exampleimage.tgz example/imagerepo
   ```

5. **删除容器**

   ```shell
   # 删除单个容器
   docker rm -f <container-id>
   
   # 清理所有处于终止状态的容器
   docker container prune
   ```

## 3.3 Dockerfile

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

```dockerfile
FROM nginx
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
```

上面的内容是一个简单的 dockerfile ，功能是基于 nginx 构建一个本地镜像。

**FROM**：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。

**RUN**：用于执行后面跟着的命令行命令。有以下俩种格式：

- **shell**

  ```shell
  RUN <shell command>
  # <shell command> 等同于，在终端操作的 shell 命令。
  ```

- **exec**

  ```
  RUN ["可执行文件", "参数1", "参数2"]
  # 例如：
  # RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
  ```

**构建镜像：**

在 Dockerfile 文件的存放目录下，执行构建动作。

```
docker build -t nginx:test .
# . 是本次执行的上下文路径，
```

**上下文路径**指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。默认上下文路径就是 dockerfile 所在的路径。

## 3.4 Compose

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

**使用步骤：**

1. 使用 Dockerfile 定义应用程序的环境。

2. 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
3. 最后，执行 docker-compose up 命令来启动并运行整个应用程序。

**docker-compose.yml 样例：**

```yaml
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

使用 docker-compose up 命令时，以依赖性顺序启动服务。在上面示例中，先启动 db 和 redis ，才会启动 web。





