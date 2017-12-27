# docker 基本概念

####  两个重要概念 image和 container
image是个轻量级，独立的，可执行文件包，其中包括运行的软件所需要的的一切，包括代码，运行时，库，环境变量和配置文件

container是image的实例化，默认情况下，它与主机环境完全隔离，只有在配置时，才访问主机文件和端口

container在主机内核上，运行应用程序，容器可以获得本地访问权限，每个容器都以独立的进程运行。

#### 虚拟机和docker相比

查看docker官方教程

虚拟机运行客户操作系统，每个vm里面，都包含了操作系统层，产生的磁盘映像和应用程序是操作系统设置，系统安装依赖关系，安全补丁。是难以管理的，如果你要开启多个虚拟机，你就必须手动配置操作环境配置环境。

容器可以共享一个内核，并且唯一需要在容器镜像中的可执行文件，以及包依赖关系，它们永远不需要安装在主机系统，这些进程像本地进程一样运行，可以通过运行命令单独管理它们，因为它们包含了多有依赖关系，没有配置上的麻烦，是一个打包后可以随处运行的应用程序。


#### 新的开发环境

Dockerfile 定义image的配置，即容器内部环境。访问网络接口，磁盘驱动器这类的资源都是在哥环境虚拟化的，这个环境与系统其他部分是隔离的，所以你必须将端口映射到外部世界，并且明确你要“复制”到那个文件到那个环境。

#### Service
服务实际上知识“生产中的容器”，服务只需要运行一个image，它规定了container，应该使用哪个端口，容器应该运行多少个副本，以便服务具有所需的容量。

在docker定义，运行服务都非常简单，只需要一个`docker-compose.yml`

`docker-compose.yml`文件是一个yaml文件，它定义了Docker容器在生产中的行为。



#### docker 在阿里云的应用
1. 开通阿里云账号
2. 创建命名空间，创建仓库
3. 然后，上传还有下载按照文档来。

### docker 入门使用

*下载ubuntu*
网络环境问题，如果阿里云的加速代理没有用的话，可以，除去掉。在Daemon这里配置，有时候加上反而不行。

下载docker
`docker run ubuntu`

docker域ubuntu交互命令，window需要前缀winpty, --interactive(或者是-i)，告诉Docker保持标准的输入六（stdin，标准输入）对容器开放，即使容器没有终端连接，--tty（或者 -t），告诉docker为容器分配一个虚拟终端，这叫允许你发信号给容器。
`winpty docker run -t -i ubuntu`

查看ubuntu内核
`uname -r`

查看ubuntu发行版本
`cat /etc/issue`

ubuntu 需要安装新的软件时，需要在image添加缓存（是否可以在DockerFile 配置文件 固定配置好软件），-qq可以省略DockerFile不需要的输出
`apt -qq update`

注意如果用`exit`命令退出container，那么这个container会被注销掉，再次`winpty docker run -t -i ubuntu`，进入ubuntu的时候，之前安装的软件都不见了。


#### docker常用命令, (因为网络环境问题，所以个别的命令可能和dockerhub不一致)
docker build -t friendlyname    # Create image using this directory's Dockerfile

docker run -p 4000:80 friendlyname  # Run "friendlyname" mapping port 4000 to 80

docker run -d -p 4000:80 friendlyname   # Same thing, but in detached mode

docker container ls # List all running containers

docker container ls -a # List all containers, even those not running

docker container stop <hash>    # Gracefully stop the specified container

docker container kill <hash> # Force shutdown of the specified container

docker container rm <hash>  # Remove specified container from this machine

docker container rm $(docker container ls -a -q)    #Remove all containers

docker image ls -a  # List all images on this machine

docker image rm <image id>  # Remove specified images on this machine

docker image rm $(docker image ls -a -q)    # Remove all images from this machine

docker login    # Login in this CLI session using your Docker credentails

docker tag <image> username/repository:tag  # Tag <image> from upload to registry

docker push username/reposity:tag # Upload tagged image to registry

docker run username/reposity:tagg   # Run image from a registry

#####  守护进程
`--detach`
在后台启动该程序，运行守护式容器适合后台默认运行的程序，这类程序被称为守护程序，守护程序通常通过网络或其他通讯工具和其他程序或者人进行交互。

##### 重新启动容器
如何程序被误停止，使用`restart`命令重启程序
docker restart web

测试程序是否运行起来，最好的办法就是检查每个容器的日志`docker logs web`会显示日志。这意味着，web服气正在运行，并且监控点正在监视该网站，这些内容将会被写入到日志。写入的日志可以持久化保存，只要该容器还存在。一个更好的方式是，使用存储卷来处理日志数据。


#### 两种发布镜像方式
1. 使用命令行来发布独立系统构建的镜像。这个方式被认为是不能信任的，因为不清楚它们究竟是如何构建的。
2. 公开Dockerfile，并使用Docker Hub的持续构建系统。Dockerfiles是构建镜像的脚本。首选自动创建的镜像版本，因为在安装之前，Dockerfile可用于检查，所以被认为可信赖的。

使用docker search搜索镜像，你可以看到镜像是从公开的脚本构建的。在AUTOMATED列寻找一个`ok`标记

#### 从Dockerfile安装
你将公共源代码库的项目复制到机器上，使用项目的Dockerfile构建Docker镜像


## 持久化存储，和存储卷间的状态共享

当初始化软件的时候，那么数据要如何保存。另一种，是不同web应用程序运行在不同的容器中，为了让他们的日志文件保持在容器之外。将如何保存，如何让其他程序访问到这些文件。这些问题的答案就是储存卷的使用。

### 储存卷的简介
一个主机或者容器的目录树是由一组挂载点创建而成，这些挂载点描述了如何鞥构建出一个或多个文件系统。存储卷是容器目录树上的挂载点，其中一部分主机目录树已经被挂载了。

#### 储存卷提供容器无关的数据管理方式
从语义上看，储存卷是一个数据分割和共享的工具，有一个与容器无关的范围或者生命周期。这是的储存卷成为了容器化系统设计中关于文件分享或写入最重要的一部分。

储存卷可以隔离应用程序和主机的关系。镜像被装载到主机，创建出一个容器。Docker不知道主机在哪里运行，只能判断哪些文件在容器中可用，这意味着docker本身没有办法利用主机上的设施，比如哦网络储存，固态硬盘。但是有主机知识的用户可以使用储存卷，在容器中将这些目录映射到主机的储存上。

##### 储存卷的类型
储存卷有两种类型，每一个储存卷就是容器目录树的挂载点在主机目录树中的位置，但不同的储存卷类型在主机的位置是不同的。第一种是绑定挂载储存卷。绑定挂载储存卷使用用户提供的主机目录或者文件。第二类型是管理储存眷顾。管理储存卷使用由docker守护进程控制的位置，被称为docker管理空间。


#### 容器的创建
创建容器有两种方法，要么手工修改现有容器的镜像，要么定义，执行一个名为Dockerfile的自动化构建脚本。

联合文件系统（UFS）挂载提供了容器的文件系统。任何对容器文件系统的改动都会被写入新的文件层中，这个文件层归创建它的容器所有。

从一个容器构建一个镜像的部分
1. 从已经存在的镜像创建一个容器
2. 修改这个容器的文件系统，这些改动回被保存在容器的联合文件系统的新文件层
3. 修改完成后，提交（commit）。一旦修改被提交了，就能从新镜像创建新的容器了。




