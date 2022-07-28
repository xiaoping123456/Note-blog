# Docker基础

## 概述

Docker是一个开源的应用容器引擎

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

### Docker的优点

Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

#### 1、快速，一致地交付您的应用程序

Docker 允许开发人员使用您提供的应用程序或服务的本地容器在标准化环境中工作，从而简化了开发的生命周期。

容器非常适合持续集成和持续交付（CI / CD）工作流程，请考虑以下示例方案：

- 您的开发人员在本地编写代码，并使用 Docker 容器与同事共享他们的工作。
- 他们使用 Docker 将其应用程序推送到测试环境中，并执行自动或手动测试。
- 当开发人员发现错误时，他们可以在开发环境中对其进行修复，然后将其重新部署到测试环境中，以进行测试和验证。
- 测试完成后，将修补程序推送给生产环境，就像将更新的镜像推送到生产环境一样简单。

#### 2、响应式部署和扩展

Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。

Docker 的可移植性和轻量级的特性，还可以使您轻松地完成动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务。

#### 3、在同一硬件上运行更多工作负载

Docker 轻巧快速。它为基于虚拟机管理程序的虚拟机提供了可行、经济、高效的替代方案，因此您可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情。

## 安装

```shell
# 1、yum 包更新到最新 
yum update
# 2、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的 
yum install -y yum-utils device-mapper-persistent-data lvm2
# 3、 设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 4、 安装docker，出现输入的界面都按 y 
yum install -y docker-ce
# 5、 查看docker版本，验证是否验证成功
docker -v

```

## 加速

使用阿里的镜像加速（免费）

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

## 常用指令

### Docker进程相关命令

- 启动docker服务

  ```shell
  systemctl start docker
  ```

- 停止docker服务

  ```shell
  systemctl stop docker
  ```

- 重启docker服务

  ```shell
  systemctl restart docker
  ```

- 查看docker服务状态

  ```shell
  systemctl status docker
  ```

- 设置开机启动docker服务

  ```shell
  systemctl enable docker
  ```

  

### Docker镜像相关命令

- 查看本地镜像

  ```shell
  docker images
  docker images -q  # 查看所有镜像的id
  ```

  ![image-20220728090104958](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728090104958.png)

  各个选项说明:

  - **REPOSITORY：**表示镜像的仓库源
  - **TAG：**镜像的标签 (同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本)
  - **IMAGE ID：**镜像ID
  - **CREATED：**镜像创建时间
  - **SIZE：**镜像大小

- 搜索镜像 ：从网络中查找需要的镜像

  ```shell
  docker search 镜像名称
  ```

- 拉取镜像 ：从Docker仓库下载镜像到本地，镜像名称格式为 `名称:版本号`，如果不指定版本号，则默认为最新版本latest。可以去docker hub 搜索查看对应的镜像

  ```shell
  docker pull 镜像名称:版本号
  ```

- 删除镜像：删除本地的镜像

  ```shell
  docker rmi 镜像id/镜像名    # 删除指定本地镜像
  docker rmi `docker images -q`   # 删除所有本地镜像
  ```

  

### Docker容器相关命令

- 查看容器

  ```shell
  docker ps    # 查看正在运行的容器
  docker ps -a    # 查看所有容器
  ```

  ![image-20220728090810716](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728090810716.png)

- 创建并启动容器

  ```shell
  docker run 参数 镜像 /bin/bash
  ```

  ![image-20220728091436916](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728091436916.png)

  参数说明：

  • -i：保持容器运行。通常与 -t 同时使用。加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。

  • -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用。

  • -d：以守护（后台）模式运行容器。创建一个容器在后台运行，需要使用docker exec 进入容器。退出后，容器不会关闭。

  • -it 创建的容器一般称为交互式容器  即exit后 容器就关闭了

  ​	-id 创建的容器一般称为守护式容器	即容器在后台启动

  • --name：为创建的容器命名。

  • -v ：设置数据卷

  • -e : 指定容器环境变量

  /bin/bash：放在镜像后是命令，这里我们希望有个交互式的Shell，因此用的是/bin/bash （不加也可以）

  如果不指定镜像的版本 默认使用latest版本

  

- 进入容器

  1. exex （推荐）使用exec进入容器后，再exit退出，容器不会关闭

     ```shell
     docker exec 参数 容器名/容器id /bin/bash
     ```

  2. attach（不推荐）   使用attach进入容器后，再exit退出，容器会自动关闭

     

- 停止容器

  ```shell
  docker stop 容器名称/id
  ```

- 启动容器

  ```
  docker start 容器名称/id
  ```

- 删除容器

  注：如果容器是运行状态则删除失败，需要停止容器才能删除

  ```shell
  docker rm 容器名称/id
  ```

- 查看容器信息

  ```shell
  docker inspect 容器名称/id
  ```

  

## Docker数据卷

Docker数据卷是为了持久化容器的数据，避免容器关闭后数据丢失。

卷就是目录或文件，存在于一个或多个容器中，由Docker挂载到容器，但卷不属于联合文件系统（Union FileSystem），因此能够绕过联合文件系统提供一些用于持续存储或共享数据的特性:。

卷的设计目的就是**数据的持久化**，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷。容器的持久化和同步操作!容器之间也是可以数据共享的!

简单来说：**数据卷是宿主机中的一个目录或文件；当容器目录和数据卷目录绑定后，对方的修改会立即同步；一个数据卷可以被多个容器同时挂载；一个容器也可以被挂载多个数据卷。**

**数据卷的作用：**

1. 容器数据持久化
2. 外部机器和容器间通信
3. 容器之间数据交换

**多容器进行数据交换**：

 1. 多个容器挂载同一个数据卷

    ![image-20220728094833152](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728094833152.png)

 2. 数据卷容器

    ![image-20220728095341770](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728095341770.png)



### 配置数据卷

创建启动容器时，使用 -v 参数 设置数据卷

数据卷挂载方式

1. 方式一：指定目录挂载

   ```shell
   docker run ... -v 宿主机目录:容器内文件目录
   
   docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql
   ```

   

2. 方式二：匿名挂载  (挂在不指定宿主机目录，也不使用卷名。)

   ```shell
   # 方式2：匿名挂载
   docker run -d --name nginx01 -v /etc/nginx nginx
   ```

   匿名挂载方式中，我们在 -v 只写了容器内的路径，没有写容器外的路径

   匿名挂载宿主机目录 ：`/var/lib/docker/volumes/卷名或id/_data` 

3. 方式三：具名挂载  （-v 自定义卷名:容器内路径）

   ```shell
   docker run -d --name nginx02 -v juming-nginx:/etc/nginx nginx
   ```

   具名挂载宿主机目录：`/var/lib/docker/volumes/卷名或id/_data`下



注意事项：

1. 目录必须是绝对路径
2. 如果目录不存在，则会自动创建
3. 可以挂载多个数据卷

多个容器可以绑定同一个数据卷



### 数据卷容器

把一个容器作为一个数据卷

创建一个容器，挂载一个目录，让其他容器继承自该容器（--volume-from）

配置数据卷容器示例：

1. 创建启动c3数据卷容器，使用 -v 参数 设置数据卷

   ```shell
   docker run –it --name=c3 –v /volume centos:7 /bin/bash
   ```

2. 创建启动c1 c2 容器，使用 `--volumes-from` 参数 设置数据卷

   ```shell
   docker run –it --name=c1 --volumes-from c3 centos:7 /bin/bash
   docker run –it --name=c2 --volumes-from c3 centos:7 /bin/bash
   ```

   

## Docker容器连接

容器中可以运行一些网络应用，要让外部也可以访问这些应用，但是

​	容器内的网络服务和外部机器不能直接通信

​	外部及其和宿主机可以直接通信

​	宿主机和容器可以直接通信

所以

​	当容器中的网路服务需要被外部机器访问时，可以将容器中提供服务的端口映射到宿主机的端口上，外部机器	访问宿主机的该端口，从而间接访问容器的服务。

通过 **-P** 或 **-p** 参数来指定端口映射。

![image-20220728103611731](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728103611731.png)

一般都是容器内服务是哪个端口映射到宿主机的哪个端口 （端口号一致）

- **-P :**是容器内部端口**随机**映射到主机的端口。
- **-p :** 是容器内部端口绑定到**指定**的主机端口。（常用）

```shell
docker run -d -p 5000:5000 training/webapp python app.py
```



## Docker应用部署

### 一、部署MySQL

1. 搜索mysql镜像

```shell
docker search mysql
```

2. 拉取mysql镜像

```shell
docker pull mysql:5.6
```

3. 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql
cd ~/mysql
```

```shell
docker run -id \
-p 3307:3306 \
--name=c_mysql \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.6
```

- 参数说明：
  - **-p 3307:3306**：将容器的 3306 端口映射到宿主机的 3307 端口。
  - **-v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。配置目录
  - **-v $PWD/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。日志目录
  - **-v $PWD/data:/var/lib/mysql** ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。数据目录
  - **-e MYSQL_ROOT_PASSWORD=123456：**初始化 root 用户的密码。



4. 进入容器，操作mysql

```shell
docker exec –it c_mysql /bin/bash
```

5. 使用外部机器连接容器中的mysql

![image-20220728112823968](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728112823968.png)

