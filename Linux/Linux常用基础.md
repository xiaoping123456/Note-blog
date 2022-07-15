# Linux常用基础

## vi-vim

### vi和vim的区别

它们都是多模式编辑器，不同的是vim 是vi的升级版本，它不仅兼容vi的所有指令，而且还有一些新的特性在里面。

### 模式

1. 正常模式（浏览模式）

2. 插入模式（修改）

   按下i,I,o,O,a,A,r,R进入

3. 命令行模式：

   按下esc，再输入 : 进入命令行模式，完成读取、存盘、替换、离开、显示行号等动作。

   常用: q（退出）、q!（强制退出）、w（保存）、wq（保存并退出）

4. 可视模式：

   在正常模式中按下v, V, `<Ctrl>+v`，可以进入可视模式。

   可视模式中的操作有点像拿鼠标进行操作，选择文本的时候有一种鼠标选择的即视感

### 常用命令

1. 拷贝当前n行  nyy  (1yy,2yy) 

2. 粘贴 p

3. 删除当前n行  ndd（1dd,2dd）

4. 查找某个单词   /关键字  （/keyword,/aa）  

   输入n  查找下一个    

   :noh 取消搜索高亮

5.  gg 光标移动到首行   

   G光标移动到最末行   

   输入n，再输入shift+g 跳转到第n行

6. 行号  :nu 显示行号

   ​          :nonu 不显示行号

   

## 用户管理

### 常用操作

1. 添加用户

   ```
   useradd 用户名
   ```

   默认添加的用户的家目录再/home/milan

   

   指定家目录

   ```
   useradd -d 指定目录 
   ```

      

2. 指定/修改密码

   ```
   passwd 用户名
   ```

3. 删除用户

   ```
   userdel 用户名
   ```

   删除用户及主目录

   ```
   userdel -r 用户名
   ```

4. 查询用户

   ```
   id 用户名
   ```

   ![image-20220708201114485](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220708201114485.png)

5. 切换用户

   ```
   su - 用户名
   ```

   注：从权限高的用户切换到权限低的用户，不需要输入密码，反之需要。

   当需要返回到原来用户时，使用 exit/logout指令

6. 查看当前用户/登录用户

   ```
   whoami/who am i
   ```

### 用户组

## 常用实用指令

### 文件目录类

1. pwd

   获取当前工作目录的绝对路径

2. ls [选项] [目录或文件]

   查看当前目录的所有信息

   常用选项：-a:显示当前目录所有的文件和目录，包括隐藏的

   ​                   -l:以列表的方式显示信息  可简写为ll

   

3. cd [参数]

   切换到指定目录

   常用：cd ../返回上一级  cd ~ 返回家目录

4. mkdir [选项] 要创建的目录

   创建目录

   常用选项：

   ​			-p ：创建多级目录

5. rmdir [选项] 要删除的空目录

   删除一个目录

   注：rmdir删除的是空目录，如果目录下有内容则无法删除。

   如果要删除非空目录 ，需要使用 

   ```
   rm -rf 目录名
   ```

6. touch 文件名称

   创建空文件 

7. cp [选项] source target

   拷贝文件到指定目录

   常用选项：

   ​		1.-r ：递归复制整个文件夹

   实例：

   ![image-20220708203208252](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220708203208252.png)

8. rm [选项] 要删除的文件或目录

   删除文件或目录

   常用选项：

   ​         -r : 递归删除整个文件家

   ​         -f : 强制删除不提示

9. mv 移动文件与目录或重命名

10. cat [选项] 文件名

    查看文件内容

    常用选项：

    ​		-n : 显示行号

11. more 文件名

    more指令是一个基于VI编辑器的文本过滤器，它以全屏幕的方式按页显示文本文件的内容。内置了若干快捷键

    ![image-20220708204339171](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220708204339171.png)

12. less 文件名

    less指令用来分屏查看文件内容，比more指令更加强大，并不是一次将整个文件加载之后才显示，而是根据显示需要加载内容，对于显示大型文件具有较高的效率

    ![image-20220708204610573](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220708204610573.png)

13. echo [选项] [输出内容]

    输出内容到控制台

    案例：![image-20220708204750043](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220708204750043.png)

14. head [选项] 文件名

    用于显示文件的开头部分内容 默认为前10行

    head -n 5 (表示查看前5行内容，5可以为任意数)

15. tail [选项] 文件名

    用于输出文件尾部的内容，默认为后10行

    tail -n 5 文件名 （表示查看后5行内容，5可以为任意数）

    tail -f 文件名 （实时追踪该文档的所有更新追加内容）

16.  `>`和`>>`

    `>`输出重定向

    `>>`追加

    ![image-20220708205325046](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220708205325046.png)

17. ln -s [原文件或目录] [软链接名]

    给原文件创建一个软链接，类似于windows里的快捷方式

    案例![image-20220708205528371](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220708205528371.png)

    注：当使用pwd时，看到的时软链接所在的目录

18. history

    查看已经执行过的历史命令

    history 10  （查看已经执行过的10条历史命令）

    

### 搜索查找类

1. find [搜索范围] [选项] 文件名或目录

   从指定目录向下递归的遍历各个子目录，将满足条件的文件或者目录显示在终端

   选项说明：

   ![image-20220708205936531](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220708205936531.png)

2. locate 要搜索的文件名

   locate 指令可以快速定位文件路径。locate指令利用事先建立的系统中所有文件名称及路径的 locate数据库实现快速定位给定的文件。Locate 指令无需遍历整个文件系统，查询速度较快。为了保证查询结果的准确度，管理员必须定期更新locate时刻

   

3. grep [选项] 从查找内容 源文件

   grep过滤查找，管道符“|”,表示将前一个命令的处理结果输出传递给后面的命令处理

   常用选项：

   ​		-n : 显示匹配行及行号

   ​		-i : 忽略字母大小写

   ​		-c : 统计符合字符串条件的行数

   

### 压缩和解压类

#### gzip/gunzip

```
gzip 文件
```

压缩文件，只能将文件压缩为*.gz文件  不能压缩文件夹

```
gunizp 文件.gz 
```

解压缩文件

#### zip/unzip

```
zip [选项] xxx.zip  要压缩的文件或目录
```

压缩文件或目录

常用选项：  -r : 递归压缩，即压缩目录

```
unzip [选项] xxx.zip 
```

解压缩文件

常用选项： -d <目录> : 指定解压后文件的存放目录

#### tar

tar指令是打包指令，最后打包后的文件是.tar.gz的文件

tar [选项] xxx.tar.gz 打包的内容 

**选项说明**

![image-20220709094003795](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220709094003795.png)

压缩文件或目录

```
tar -zcvf testdir/
```

解压文件到当前目录

```
tar -zxvf testdir.tar.gz 
```

解压文件到指定目录

```
tar -zxvf testdir.tar.gz -C testdir2
```

## 进程管理

Linux中，每个执行的程序都称为一个进程 每一个进程进程都分配一个ID号（pid,进程号）

可以理解为：程序是静态的 进程是运行的程序



### ps指令 查看进程信息

#### ps -aux

```
ps -aux | grep xxx                            查看当前用户后台有没有xxx进程

// -a :显示当前终端的执行的所有进程信息
// -u :以用户的格式显示进程信息
// -x :显示后台进程运行的参数
// 常用 -aux
```

#### ![image-20220709101421868](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220709101421868.png)

![image-20220709100224090](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220709100224090.png)

#### ps -ef

```
ps -ef 是以全格式显示当前所有的进程
// -e :显示所有进程
// -f :全格式
```

![image-20220709103022734](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220709103022734.png)

![image-20220709103007413](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220709103007413.png)

#### -aux 和 -ef的区别

1. -ef是BSD风格  -aux是System V展示风格
2. COMMADN列如果过长，`aux`会截断显示，而`ef`不会

如果想查看进程的CPU占用率和内存占用率，可以使用`aux` 

如果想查看进程的父进程ID和完整的COMMAND命令，可以使用`ef`

### 终止进程

```
kill [选项] 进程号          //通过进程号杀死进程
killall 进程名称		    //通过进程名称杀死进程，支持通配符

常用选项：
	-9 ：强制杀死某进程
```

### 进程树pstree

```
pstree [选项]				//查看进程树，更直观的看进程信息

// -p : 显示进程的PID
// -u : 显示进程的所属用户
```

### 动态监控进程top

top和ps类似，最大的不同之处在于，top在执行一段时间可以更新正在运行的进程。

```
top [选项]

选项说明
	-d 秒数 : 指定top命令每隔几秒更新。默认是3秒
	-i : 使top不显示任何闲置或者僵尸进程
	-p : 通过指定监控进程ID来仅仅监控某个进程的状态
```

![image-20220709114326061](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220709114326061.png)

#### 交互操作

进入top后  输入 P  以CPU使用率排序，默认

​							M 以内存的使用率排序

​							N 以PID排序

​							q 退出top

### 监控网络状态

#### 查看系统网络情况 netstat

```
netstat [选项]

选项说明：
	-an : 按一定顺序排列输出
	-p : 显示哪个进程在调用
	-a : 列出所有端口
	-at : 列出所有TCP端口
	-au : 列出所有UDP端口
	-s : 显示所有端口的统计信息
```

```
netstat [选项] | grep 端口号/服务名    //查看某端口或服务的网络情况 (使用grep过滤)
```

例：![image-20220709115859168](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220709115859168.png)



## 服务管理

### systemctl

centos7.0之后 更多使用systemctl

```
systemctl [start|stop|restart|status] 服务名
```

systemctl 指令管理的服务在 /usr/lib/systemd/system 查看

#### systemctl设置服务的自启动状态

```
systemctl list-unit-files [|grep 服务名]  			//查看服务的开机启动状态
systemctl enable 服务名							//设置服务卡机启动
systemctl disable 服务名							//关闭服务开机启动
systemctl is-enabled 服务名						//查询某个服务是否是自启动的
```

### 防火墙和端口设置

#### 防火墙设置

```
systemctl start firewalld.service            #启动防火墙 
systemctl stop firewalld.service             #停止防火墙 
systemctl status firewalld                   #查看防火墙状态
systemctl enable firewalld                   #设置防火墙随系统启动
systemctl disable firewalld                  #禁止防火墙随系统启动
firewall-cmd --state                         #查看防火墙状态 
firewall-cmd --reload                        #更新防火墙规则  
firewall-cmd --list-ports                    #查看所有打开的端口 
firewall-cmd --list-services                 #查看所有允许的服务 
firewall-cmd --get-services                  #获取所有支持的服务 
```

#### 端口控制

```
firewall-cmd --query-port=8080/tcp                       # 查询端口是否开放
firewall-cmd --add-port=8080/tcp --permanent             #永久添加8080端口例外(全局)
firewall-cmd --remove-port=8800/tcp --permanent          #永久删除8080端口例外(全局)
firewall-cmd --add-port=65001-65010/tcp --permanent      #永久增加65001-65010例外(全局) 
firewall-cmd  --zone=public --add-port=8080/tcp --permanent            #永久添加8080端口例外(区域public)
firewall-cmd  --zone=public --remove-port=8080/tcp --permanent         #永久删除8080端口例外(区域public)
firewall-cmd  --zone=public --add-port=65001-65010/tcp --permanent     #永久增加65001-65010例外(区域public)
```

#### 命令解析

```
firwall-cmd：是Linux提供的操作firewall的一个工具（服务）命令

--zone #作用域
--add-port=8080/tcp #添加端口，格式为：端口/通讯协议 ；add表示添加，remove则对应移除
--permanent #永久生效，没有此参数重启后失效
```

## RPM 与 YUM

### RPM

rpm用于互联网下载包的打包及安装工具

**查询指令 **

```
rpm -qa    //查询已安装的rpm列表
rpm -q 软件包名		//查询该软件包是否安装
rpm -qi 软件包名	//从查询软件包信息
rpm -ql 软件包名	//查询软件包中的文件
rpm -qf 文件全路径名	//查询文件所属的软件包
```

**卸载rpm**

```
rpm -e RPM包的名称
```

如果其他软件包依赖于您要卸载的软件包，卸载时则会产生错误信息

可以添加参数 --nodeps，进行强制卸载（不推荐）

**安装rpm**

```
rpm -ivh RPM包全路径的名称

参数说明：
	i=install 安装
	v=verbose 提示
	h=hash 进度条
```

### YUM

yum是一个Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载rpm包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包。

**查询指令**

```
yum list | grep xx 		//查询yum服务器中拥有的需要安装的软件列表
```

**安装**

```
yum install xxx			//下载安装
```

## 文件下载

### wget

wget命令是Linux系统用于从Web下载文件的命令行工具，支持 HTTP、HTTPS及FTP协议下载文件，而且wget还提供了很多选项，例如下载多个文件、后台下载，使用代理等等。

```
wget [选项] [url]

示例：
	wget https://download.redis.io/releases/redis-6.0.8.tar.gz
	该命令会下载文件到当前工作目录中，在下载过程中，会显示进度条、文件大小、下载速度等。
```

选项说明：

```
1. 使用 -O 选项以其他名称保存下载的文件
	要以其他名称保存下载的文件，使用-O选项，后跟指定名称即可：
	wget -O redis.tar.gz https://download.redis.io/releases/redis-6.0.8.tar.gz
2. 使用 -P 选项将文件下载到指定目录
	默认情况下，wget将下载的文件保存在当前工作目录中，使用-P选项可以将文件保存到指定目录下，例如，下面将将文件下载到/usr/software目录下：
	wget -P /usr/software https://download.redis.io/releases/redis-6.0.8.tar.gz
3. 使用 -c 选项断点续传
	当我们下载一个大文件时，如果中途网络断开导致没有下载完成，我们就可以使用命令的-c选项恢复下载，让下载从断点续传，无需从头下载。
	wget -c https://download.redis.io/releases/redis-6.0.8.tar.gz
4. 使用 -b 选项在后台下载
	默认情况下，下载过程日志重定向到当前目录中的wget-log文件中，要查看下载状态，可以使用tail -f wget-log查看。
	wget -b https://download.redis.io/releases/redis-6.0.8.tar.gz
5. 使用 -i 选项下载多个文件
	如果先要一次下载多个文件，首先需要创建一个文本文件，并将所有的url添加到该文件中，每个url都必须是单独的一行。
	vim download_list.txt
	然后使用-i选项，后跟该文本文件：
	wget -i download_list.txt
6. 使用 --limit-rate 选项限制下载速度
	默认情况下，wget命令会以全速下载，但是有时下载一个非常大的资源的话，可能会占用大量的可用带宽，影响其他使用网络的任务，这时就要限制下载速度，可以使用--limit-rate选项。例如，以下命令将下载速度限制为1m/s：
	wget --limit-rate=1m https://download.redis.io/releases/redis-6.0.8.tar.gz
7. 使用 -U 选项设定模拟下载
	如果远程服务器阻止wget下载资源，我们可以通过-U选项模拟浏览器进行下载，例如下面模拟谷歌浏览器下载。
	wget -U 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.43 Safari/537.36' https://download.redis.io/releases/redis-6.0.8.tar.gz
8. 使用 --tries 选项增加重试次数
	如果网络有问题或下载一个大文件有可能会下载失败，wget默认重试20次，我们可以使用-tries选项来增加重试次数。
	wget --tries=40 https://download.redis.io/releases/redis-6.0.8.tar.gz
```

