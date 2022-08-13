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

### 示例：部署MySQL

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

## Dockerfile

### Docker镜像原理

docker镜像是由特殊的文件系统叠加而成

最低端是bootfs，并使用宿主机的bootfs

第二层是root文件系统rootfs，称为base image（基础镜像）

然后再往上可以叠加其他的镜像文件

统一文件系统技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统

一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像成为基础镜像。

当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器

![image-20220728221056597](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728221056597.png)

**灵魂三问**

1. Docker 镜像本质是什么？

   是一个分层文件系统

2. Docker 中一个centos镜像为什么只有200MB，而一个centos操作系统的iso文件要几个G？

   Centos的iso镜像文件包含bootfs和rootfs，而docker的centos镜像复用操作系统的bootfs，只有rootfs和其他镜像层

3. Docker 中一个tomcat镜像为什么有500MB，而一个tomcat安装包只有70多MB？

   由于docker中镜像是分层的，tomcat虽然只有70多MB，但他需要依赖于父镜像和基础镜像，所有整个对外暴露的tomcat镜像大小500多MB

### Dockerfile概念

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。每一条指令构建一层镜像，因此每一条指令的内容，就是描述该层镜像应当如何构建。

- 对于开发人员：可以为开发团队提供一个完全一致的开发环境

- 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了

- 对于运维人员：在部署时，可以实现应用的无缝移植

**Dockerfile的格式**

已#开头为注释行

由专用“指令（Instruction）”开头的指令行

**常用指令**

| 关键字      | 作用                     | 备注                                                         |
| ----------- | ------------------------ | ------------------------------------------------------------ |
| FROM        | 指定父镜像               | 指定dockerfile基于那个image构建                              |
| MAINTAINER  | 作者信息                 | 用来标明这个dockerfile谁写的                                 |
| LABEL       | 标签                     | 用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看 |
| RUN         | 执行命令                 | 执行一段命令 默认是/bin/sh 格式: RUN command 或者 RUN ["command" , "param1","param2"] |
| CMD         | 容器启动命令             | 提供启动容器时候的默认命令 和ENTRYPOINT配合使用.格式 CMD command param1 param2 或者 CMD ["command" , "param1","param2"] |
| ENTRYPOINT  | 入口                     | 一般在制作一些执行就关闭的容器中会使用                       |
| COPY        | 复制文件                 | build的时候复制文件到image中                                 |
| ADD         | 添加文件                 | build的时候添加文件到image中 不仅仅局限于当前build上下文 可以来源于远程服务 |
| ENV         | 环境变量                 | 指定build时候的环境变量 可以在启动的容器的时候 通过-e覆盖 格式ENV name=value |
| ARG         | 构建参数                 | 构建参数 只在构建的时候使用的参数 如果有ENV 那么ENV的相同名字的值始终覆盖arg的参数 |
| VOLUME      | 定义外部可以挂载的数据卷 | 指定build的image那些目录可以启动的时候挂载到文件系统中 启动容器的时候使用 -v 绑定 格式 VOLUME ["目录"] |
| EXPOSE      | 暴露端口                 | 定义容器运行的时候监听的端口 启动容器的使用-p来绑定暴露端口 格式: EXPOSE 8080 或者 EXPOSE 8080/udp |
| WORKDIR     | 工作目录                 | 指定容器内部的工作目录 如果没有创建则自动创建 如果指定/ 使用的是绝对地址 如果不是/开头那么是在上一条workdir的路径的相对路径 |
| USER        | 指定执行用户             | 指定build或者启动的时候 用户 在RUN CMD ENTRYPONT执行的时候的用户 |
| HEALTHCHECK | 健康检查                 | 指定监测当前容器的健康监测的命令 基本上没用 因为很多时候 应用本身有健康监测机制 |
| ONBUILD     | 触发器                   | 当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后 会执行 ONBUILD的命令 但是不影响当前镜像 用处也不怎么大 |
| STOPSIGNAL  | 发送信号量到宿主机       | 该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出。       |
| SHELL       | 指定执行脚本的shell      | 指定RUN CMD ENTRYPOINT 执行命令的时候 使用的shell            |

- FROM

  指定基础镜像，必须为第一个指令

  ```
  格式：   
     FROM <image>
  　　FROM <image>:<tag>
  　　FROM <image>@<digest>
  
  示例：　　
  	FROM mysql:5.6
  注：
     tag或digest是可选的，如果不使用这两个值时，会使用latest版本的基础镜像
  ```

- MAINTAINER

  维护者信息 （也可以不写）

  ```
  格式：
      MAINTAINER <name>
  示例：
      MAINTAINER bertwu
      MAINTAINER xxx@163.com
      MAINTAINER bertwu <xxx@163.com>
  ```

- RUN

  构建镜像时执行的命令，有两种执行方式

  ```
  shell执行
  格式：
      RUN <command>
  exec执行
  格式：
      RUN ["executable", "param1", "param2"]
  示例：
      RUN ["executable", "param1", "param2"]
      RUN apk update
      RUN ["/etc/execfile", "arg1", "arg1"]
  注：RUN指令创建的中间镜像会被缓存，并会在下次构建中使用。如果不想使用这些缓存镜像，
  可以在构建时指定--no-cache参数，如：docker build --no-cache
  ```

- ADD

  将本地文件添加到容器中，tar类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似wget

  ```
  格式：
      ADD <src>... <dest>
      ADD ["<src>",... "<dest>"] 用于支持包含空格的路径
  示例：
      ADD hom* /mydir/          # 添加所有以"hom"开头的文件
      ADD hom?.txt /mydir/      # ? 替代一个单字符,例如："home.txt"
      ADD test relativeDir/     # 添加 "test" 到 `WORKDIR`/relativeDir/
      ADD test /absoluteDir/    # 添加 "test" 到 /absoluteDir/
  ```

  

- CMD

  构建镜像后调用，也就是在容器启动时才进行调用。

  ```
  格式：
      CMD ["executable","param1","param2"] (执行可执行文件，优先)
      CMD ["param1","param2"] (设置了ENTRYPOINT，则直接调用ENTRYPOINT添加参数)
      CMD command param1 param2 (执行shell内部命令)
  示例：
      CMD echo "This is a test." | wc -l
      CMD ["/usr/bin/wc","--help"]
  
  注：CMD不同于RUN，CMD用于指定在容器启动时所要执行的命令，而RUN用于指定镜像构建时所要执行的命令。
  ```

  

- WORKDIR

  工作目录，类似于cd命令

  ```
  格式：
      WORKDIR /path/to/workdir
  示例：
      WORKDIR /a  (这时工作目录为/a)
      WORKDIR b  (这时工作目录为/a/b)
      WORKDIR c  (这时工作目录为/a/b/c)
  注：　
    通过WORKDIR设置工作目录后，Dockerfile中其后的命令RUN、CMD、ENTRYPOINT、ADD、COPY
    等命令都会在该目录下执行。在使用docker run运行容器时，可以通过-w参数覆盖构建时所设置的工作目录。
  ```

  

- 

### 制作镜像

#### 案例一：制作centos镜像

下面通过编写Dockerfile文件来制作Centos镜像，并在官方镜像的基础上添加vim和net-tools工具。首先在/home/dockfile 目录下新建文件Dockerfile。然后使用上述指令编写该文件。

```shell
Dockerfile
[root@localhost dockerfile]# cat Dockerfile 
FROM centos:7
MAINTAINER bertwu <1258398543@qq.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim   net-tools
EXPOSE 80
CMD /bin/bash
```

逐行解释该Dockerfile文件的指令：

FROM centos:7 该image文件继承官方的centos7
ENV MYPATH /usr/local：设置环境变量MYPATH
WORKDIR $MYPATH：直接使用上面设置的环境变量，指定/usr/local为工作目录
RUN yum -y install vim && net-tools：在/usr/local目录下，运行yum -y install vim和yum -y install net-tools命令安装工具，注意安装后的所有依赖和工具都会打包到image文件中
EXPOSE 80：将容器80端口暴露出来，允许外部连接这个端口
CMD：指定容器启动的时候运行命令
下面执行build命令生成image文件，如果执行成功，可以通过docker images来查看新生成的镜像文件。

```shell
[root@localhost dockerfile]# docker build -t mycentos:1.0 . 

[root@localhost dockerfile]# docker images
REPOSITORY    TAG             IMAGE ID       CREATED              SIZE
mycentos      1.0             e0316e2ed3a5   About a minute ago   409MB

```

可以使用 `docker history 镜像id` 查看镜像构建过程

```
[root@localhost dockerfile]# docker history  e0316e2ed3a5 
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
e0316e2ed3a5   2 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B        
79738577ded0   2 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
f10acdc62daf   2 minutes ago   /bin/sh -c yum -y install vim   net-tools       205MB     
40b0252c02c7   3 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0B        
d38940eb3b75   3 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B        
b23dc50b92b4   3 minutes ago   /bin/sh -c #(nop)  MAINTAINER bertwu <125839…   0B        
eeb6ee3f44bd   2 months ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      2 months ago    /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      2 months ago    /bin/sh -c #(nop) ADD file:b3ebbe8bd304723d4…   204MB    

```

#### 案例二：制作springboot应用镜像

```
cat Dockerfile

FROM openjdk:8-jre # jar包基于jdk ,war包基于tomcat
WORKDIR /app
ADD demo-0.0.1-SNAPSHOT.jar app.jar # 将上下文中 jar包复制到 /app目录下，并且重命名为app.jar
EXPOSE 8081 # 暴露端口
ENTRYPOINT[ "java" , "-jar" ] # 启动应用固定命令
CMD["app.jar"] # 动态传递jar包名
```

![image-20220728225602809](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220728225602809.png)

## Docker Compose

### 概述

Docker Compose是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止。使用步骤：

1. 利用 Dockerfile 定义运行环境镜像
2. 使用 docker-compose.yml 定义组成应用的各服务
3. 运行 docker-compose up 启动应用

### 安装Docke Compose

```shell
# Compose目前已经完全支持Linux、Mac OS和Windows，在我们安装Compose之前，需要先安装Docker。下面我 们以编译好的二进制包方式安装在Linux系统中。 
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 设置文件可执行权限 
chmod +x /usr/local/bin/docker-compose
# 查看版本信息 
docker-compose -version
```

卸载

```shell
# 二进制包方式安装的，删除二进制文件即可
rm /usr/local/bin/docker-compose
```

### 案例：使用docker compose编排nginx+springboot项目

1. 创建docker-compose目录

```shell
mkdir ~/docker-compose
cd ~/docker-compose
```

2. 编写 docker-compose.yml 文件

```shell
version: '3'
services:
  nginx:
   image: nginx
   ports:
    - 80:80
   links:
    - app
   volumes:
    - ./nginx/conf.d:/etc/nginx/conf.d
  app:
    image: app
    expose:
      - "8080"
```

3. 创建./nginx/conf.d目录

```shell
mkdir -p ./nginx/conf.d
```



4. 在./nginx/conf.d目录下 编写itheima.conf文件

```shell
server {
    listen 80;
    access_log off;

    location / {
        proxy_pass http://app:8080;
    }
   
}
```

5. 在~/docker-compose 目录下 使用docker-compose 启动容器

```shell
docker-compose up
```

6. 测试访问

```shell
http://192.168.149.135/hello
```

