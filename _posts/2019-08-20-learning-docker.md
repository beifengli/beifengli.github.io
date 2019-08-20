---
title:  "docker 基础"
header:
categories: 
  - program
tags:
  - linux
  - docker 
---
Docker 是一种新型的虚拟化方式，与传统的虚拟化方式相比有着诸多优势。Docker启动时间快，能够提供一致的开发和测试环境，能够实现持续集成、持续交付、部署，能够方便的进行维护和拓展。

### 对比总结

特性|Docker|虚拟机
:--:|:--:|:--:  
启动|秒级|分钟级
硬盘使用|MB|GB|
性能|接近原生|弱于原生
系统支持量|单机支持上千个|一般几十个

<!--more-->
## 一、docker三要素

* 镜像 image
* 容器 container
* 仓库 repository

序号|docker|C++|含义
:--:|:--:|:--:|:--
1|image|类|镜像是一个**只读**的模板，可以用来创建docker容器，一个镜像可以创建多个容器。
2|container|对象|容器是镜像的实例，利用容器运行的一个或是一组应用。每个容器相互隔离、保证安全的平台。可看成简易版的linux环境和运行在其中的应用程序。容器可读可写的。
3|repository|仓库（git)|集中存放镜像文件的场所，仓库分为公有仓库和私有仓库。

Docker本身是一个容器运行载体或者称之为管理引擎。我们把应用程序和配置依赖打包形成一个可交付的运行环境，这个打包好的运行环境就是image镜像文件。只有通过这个镜像文件才能生成Docker容器。image文件可以看做是容器的模板。Docker根据image文件生成容器的实例。同一个image文件，可以生成多个同时运行的容器实例。

* image文件生成的容器实例，本身也是一个文件，是镜像文件。
* 一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例。
* 仓库，是存放了一堆镜像的地方，可以将镜像发布到仓库中，需要的时候拉下了就是了。

## 二、安装  

### 2.1 centOS 6.8 安装

>yum install -y epel-release  
yum install -y docker-io  

安装后的配置文件：/etc/sysconfig/docker

>
other_args=  
DOCKER_CERT_PATH=/etc/docker  
DOCKER_NOWARN_KERNEL_VERSION=1

启动docker服务： service docker start

验证：docker version

### 2.2 centOS 7 安装

<https://docker-cn.com>

### 2.3 ubuntu 安装

#### 2.3.1 使用apt安装

```#! /bin/bash
sudo apt-get update
sudo apt-get install docker-ce
```

#### 2.3.2 使用脚本安装

Docker官方为了简化安装流程，提供了一套边界的安装脚本。

```#! /bin/bash
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

### 2.4 镜像加速

[阿里云镜像加速](https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fcr.console.aliyun.com%2Fcn-hangzhou%2Fmirrors)  
[Azure中国镜像](https://github.com/Azure/container-service-for-azure-china/blob/master/aks/README.md#22-container-registry-proxy)

在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）

```#! /bin/bash
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com"
  ]
}
```

### 2.5 启动Docker

```#! /bin/bash
sudo systemctl enable docker
sudo systemctl start docker
```

### 2.6 建立Docker用户组

```#! /bin/bash
#建立docker用户组
sudo groupadd docker  
#将当前用户加入docker组
sudo usermod -aG docker $USER 
```

### 2.7 验证是否安装正确

```#! /bin/bash
docker run hello-world
```

## 三、原理

### 底层原理

#### 虚拟机和Docker

Docker没有硬件虚拟层

## 四、Docker常用命令  

### 4.1 帮助命令

```#! /bin/bash
docker version  //查看版本
docker info     //查看详细信息
docker  --help  //查看帮助
```

### 4.2 镜像命令

#### 1. 列出镜像

```#! /bin/bash
 docker images  //列出本地image
```

序号|参数|含义
:--:|:--:|:--:
1|-a|列出本地所有镜像
2|-q|显示当前镜像的ID
3|-digest|显示说明
4|--no-trunc|没有节选

#### 2. 查找镜像

```#! /bin/bash
docker search name  ///查找镜像
```

序号|参数|含义
:--:|:--:|:--:
1|--no-trunc|没有节选
2|-s|列出收藏数不小于指定值的镜像  
3|--automated|只列出automated build类型的镜像  

#### 3. 下载镜像

> docker pull name:tags  //下载镜像

#### 4. 删除镜像

> docker rmi nameid     //删除镜像

删除单个  docker rmi -f nameid

删除多个  docker rmi -f nameid1 nameid2

删除全部  docker rmi -r $(docker images -qa)

### 4.3 容器命令

#### 1. 启动容器

```#! /bin/bash
docker run [option] image [command] [ARG]
docker run -it
```

序号|参数|含义
:--:|:--:|:--:
1|-i|交互式启动
2|-t|创建一个伪终端  
3|-d|在后台运行容器  
4|-P|随机端口映射  
5|-p|指定端口映射 hostport:containerport  

#### 2. 列出运行容器

> docker ps

序号|参数|含义
:--:|:--:|:--:
1|-l|上一个容器  
2|-a|所有容器  
3|-n|显示最近n个创建的容器  
4|-q|静默模式，只显示容器编号  
5|--no-trunc|不截断输出  

#### 3. 退出容器

>exit //容器停止退出  
ctrl+P+Q //容器不停止退出

#### 4. 停止容器

> docker stop id  // 缓慢停止  
docker kill    //缓慢强制

#### 5. 重启容器

> docker restart  id

#### 6. 删除容器

>docker rm //删除容器  
docker rmi //删除镜像  

* 一次性删除多个容器

>docker rm -f $(docker ps -a -q)  
docker ps -a -q | xargs docker rm  

## 五、重要命令  

### 1. 守护容器  

> docker run -d centos

* docker后台运行，就必须有一个前台进程。

> docker run -d centos /bin/bash -c "while true;do echo hello zzyy;sleep 2;done"  
> //持续输出hello zzyy

### 2. 查看容器内进程

> docker top id  
> docker exec -t id [命令]  
docker exec -t id /bin/bash

exec 既可以在外面得到运行内的结果，也可以进入容器内部进行命令处理

### 3. 将容器内的文件复制到宿主主机

> docker cp id:路径  路径
> docker cp id:/tmp/yum.log /root

## 六、镜像原理

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

### 1. 联合文件系统

UnionFS (联合文件系统)自Union文件系统(UnionFS) 是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为次提交来层层的叠 加，同时可以将不同目录挂载到同一个虚拟文件系统F(unite several directories into a single virtual flesystem)。

Union文件系统是Docker镜象的基础。镜像可以通过分层来进行继承，基于基础镜像(没有父镜像)，可以制作各种具体的应用镜像.

特征：一次同时加载多个文件系统，但从外面看来，只能看到一个文件系统，联合加载起来，这样最终的文件系统会包含所有底层的文件和目录。

### 2. Docker 镜像加载原理

docker的镜像实际上是由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包好bootloader和kernel,bootloader主要是引导加载kernel,linux刚启动时会加载bootfs文件系统，在docker镜像的最底层是bootfs。这一层与我们典型的linux/unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已有bootfs转交给内核，此时系统也会卸载bootfs。

rootfs(root file system),在bootfs之上。包含的是典型linux系统中的/dev,/proc,/bin,/etc等标准目录和文件。rootfs就是各种不同的操作系统发型版，比如Ubuntu，centos等。

对于一个精简版的OS，rootfs可以很小，只需要包括最基本的命令，工具和程序库就可以，因为底层直接用Host的kernel，自己只需要提供rootfs就好了。由此可见对于不同的linux发行版，bootfs基本是一致的，rootfs会有所差别，因此不同的发行版可以共用bootfs。

### 3. 分层的镜像

采用分层镜像的原因：共享资源  

当有多个镜像都从相同的base镜像中构建而来，那么宿主机只需要在磁盘上保存一份base镜像，同时内存中也只需要加载一份base镜像，就可以为所有容器服务。

镜像的每一层都可以被共享。

#### 特点

docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”,“容器层”之下的都叫“镜像层”。

## 七、docker commit操作

docker commit 提交容器副本使之成为一个新的镜像。

>docker commit -m=“提交的描述信息” -a="作者“ 容器ID 要创建的目标镜像名：[标签名]

## 八、容器数据卷

### 1. docker 的理念：

* 将运用与运行的环境打包形成容器运行，运行可以伴随着容器，但是我们对数据的要求希望是持久化的。
* 容器之间希望有可能共享数据。

docker 容器产生的数据，如果不通过docker commit生成新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了。

目的：数据持久化,数据共享。

### 2. 数据卷

卷就是目录或文件，存在与一个或多个容器中，由docker挂载到容器中，但不属于联合文件系统，因此能够绕过Union File System 提供一些用于持续存储或共享数据的特性：

卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会再容器删除时删除其挂载的数据卷。

### 3. 特点

1. 数据卷可以在容器之间共享或重用数据  
2. 卷中的更改可以直接生效  
3. 数据卷中的更改不会包含在镜像的更新中  
4. 数据卷的生命周期一直持续到没有容器使用它为止  
5. 容器到主机、主机到容器之间的数据共享

## 九、容器数据卷的使用

### 1.命令添加

> docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名
docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名  //只读

### 2. Dockrfile 添加

#### 1. Dockerfile

镜像的源码描述

加载数据卷

VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]

> FROM centos
VOLUME ["dataVolumeContainer1","/dataVolumeContainer2"]
CMD echo "finished,-------seccess1"
CMD /bin/bash

#### 2. 生成镜像

> docker build -f /mydocker/Dockerfile -t zzyy/centos .

#### 3. 默认加载卷

> /var/lib/docker/volumes/.../_data

#### 4. 挂载报错

Docker 挂载主机目录Docker访问出现cannot open directory : Permission denied

解决办法：在挂载目录后多加一个 --privileged=true 参数即可

## 十、数据卷容器

命名的容器挂载数据卷，其他容器通过挂载这个（父容器）实现数据共享，挂载数据卷的容器，称之为数据卷容器。

> docker run -it --name  dc02 --volume-form dc01 zzyy/centos

容器之间配置信息的传递，数据卷的什么周期一直持续到没有容器使用它为止。

## 十一、Dockerfile

步骤：

1. 手动编写Dockerfile,必须符合Dockerfile 编写规则
2. 有这个文件后，直接docker build执行，获得一个自定义的镜像
3. run

基础镜像：scratch

### 1. Dockerfile 构建过程

基本规则：

1. 每条保留字指令都必须为大写字母，且后面要跟谁至少一个参数
2. 指令按照从上到下，顺序执行
3. #表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

执行流程：

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 指令类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令，直到所有指令都执行完成

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同的阶段。

* Dockerfile 是软件的原材料
* Docker 镜像是软件的交付品
* Docker 容器是软件的运行态

Dockerfile 面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

1. Dockerfile，需要定义一个Dockerfile,Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进行和内核进程（当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制）等等；
2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个docker镜像，当运行docker镜像时，会真正开始提供服务；
3. Docker容器，容器是直接提供服务的。

### 2. docker保留字指令

序号|关键词|含义
:--:|:--:|:--
1|FROM|基础镜像
2|MAINTAINER|镜像维护者的姓名和邮箱地址
3|RUN|容器构建时需要运行的命令
4|EXPOSE|当前容器对外暴露的端口
5|WORKDIR|指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
6|EVN|用来在构建镜像过程中设置环境变量
7|ADD|将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
8|COPY|类似ADD,拷贝文件和目录到镜像中。将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置  
9|VOLUME|容器数据卷，用于数据持久化和保存
10|CMD|指定一个容器启动时要运行的命令；Dockerfile中可以有多个CMD指令，**但只有最后一个生效**，CMD会被docker run之后的参数替换
11|ENTRYPOINT|指定一个容器启动时要运行的命令，ENTRYPOINT的目的和CMD一样，都是在制定容器启动程序和参数
12|ONBUILD|当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

COPY 命令

>COPY src dest
COPY ["src","dest']

CMD 容器启动命令

> CMD <命令>  
CMD ["可执行文件","参数1","参数2",...]  
CMD ["参数1","参数2"...]   //在指定了ENTRYPOINT 指令后，用CMD指定具体的参数  

### 3. 案例

BASE镜像scratch

#### 1. 案例1

需求：

1. 指定默认路径
2. vim编辑器
3. 配置网络

>FROM centos  
ENV mypath /tmp  
WORKDIR $mypath  
RUN yum -y install vim  
RUN yum -y install net-tools  
EXPOSE 80
CMD /bin/bash

#### 2. 案例2

curl 命令可以用来执行下载、发送各种HTTP请求，指定HTTP头部等操作。
如果系统没有curl可以使用yum install curl 安装，也可以下载安装。

使用命令：curl <http://www.baidu.com>

执行后，www.baidu.com的html就会显示在屏幕上了
