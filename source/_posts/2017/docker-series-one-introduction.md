---
title: Docker系列之一：初识Docker
date: 2017-05-26 05:58:49
categories:
  - 容器技术
tags:
  - Docker
---
近些年容器这个技术都比较火，刚好前端时间我们新项目的设计时就选用的容器+微服务的架构，因此在这记录下Docker的部署方法。

<!-- more -->

### 1. Docker简介
Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

#### 1.1. Docker VS VM
从下图可以看出，VM是一个运行在宿主机之上的完整的操作系统，VM运行自身操作系统会占用较多的CPU、内存、硬盘资源。Docker不同于VM，只包含应用程序以及依赖库，基于libcontainer运行在宿主机上，并处于一个隔离的环境中，这使得Docker更加轻量高效，启动容器只需几秒钟之内完成。由于Docker轻量、资源占用少，使得Docker可以轻易的应用到构建标准化的应用中。但Docker目前还不够完善，比如隔离效果不如VM，共享宿主机操作系统的一些基础库等；网络配置功能相对简单，主要以桥接方式为主；查看日志也不够方便灵活。
![Docker VS VM](https://wx3.sinaimg.cn/mw690/4ca4c33cly1ffyplvxi7cj21hy0xkju5.jpg "Docker VS VM")
另外，IBM发表了一篇关于虚拟机和Linux container性能对比的论文，论文中实际测试了虚拟机和Linux container在CPU、内存、存储IO以及网络的负载情况，结果显示Docker容器本身几乎没有什么开销，但是使用AUFS会一定的性能损耗，不如使用Docker Volume，Docker的NAT在较高网络数据传输中会引入较大的工作负载，带来额外的开销。不过container的性能与native相差不多，各方面的性能都一般等于或者优于虚拟机。Container和虚拟机在IO密集的应用中都需要调整优化以更好的支持IO操作，两者在IO密集型的应用中都应该谨慎使用。

#### 1.2. Docker主要特性
1. 文件系统隔离：每个进程容器运行在完全独立的根文件系统里。
2. 资源隔离：可以使用cgroup为每个进程容器分配不同的系统资源，例如CPU和内存。
3. 网络隔离：每个进程容器运行在自己的网络命名空间里，拥有自己的虚拟接口和IP地址。
4. 写时复制：采用写时复制方式创建根文件系统，这让部署变得极其快捷，并且节省内存和硬盘空间。
5. 日志记录：Docker将会收集和记录每个进程容器的标准流（stdout/stderr/stdin），用于实时检索或批量检索。
6. 变更管理：容器文件系统的变更可以提交到新的映像中，并可重复使用以创建更多的容器。无需使用模板或手动配置。
7. 交互式Shell：Docker可以分配一个虚拟终端并关联到任何容器的标准输入上，例如运行一个一次性交互shell。

#### 1.3. Docker主要应用场景
- web应用的自动化打包和发布；
- 自动化测试和持续集成、发布；
- 在服务型环境中部署和调整数据库或其他的后台应用；
- 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。

### 2. 搭建Docker环境
最新版本的Docker采用了年份+月份的版本发布规则，分为免费和商用两个版本（Docker-CE为免费版本/Docker-EE为商用版本）。而且Docker-CE仅支持64bit的系统。
以下教程为在CentOS7的64bit系统中使用yum命令下载安装Docker。
**安装docker仓库**
```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
**获取最新版本的Docker CE**
```bash
sudo yum -y install docker-ce
```
**启动Docker进程**
```bash
sudo systemctl start docker
```
**测试Docker**
```bash
sudo docker run hello-world
```
运行hello-world后如果控制台输出如下语句则表示Docker已经安装成功。
```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```
### 3. Docker简单命令
**查询Docker Hub中的镜像**
```bash
$ docker search java
NAME            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
java            Java is a concurrent, class-based, and obj...   1368      [OK]
ibmjava         Official IBM® SDK, Java™ Technology Editio...   27        [OK]

```
**下载Docker Hub中的镜像**
```bash
$ docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
78445dd45222: Already exists 
Digest: sha256:c5515758d4c5e1e838e9cd307f6c6a0d620b5e07e6f927b07d05f6d12a1ac8d7
Status: Downloaded newer image for hello-world:latest
```
**查看已下载的所有镜像**
```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
java                8                   d23bdf5b1b1b        4 months ago        643 MB
hello-world         latest              48b5124b2768        4 months ago        1.84 kB
```
**运行一个新的容器**
```bash
docker run -itd \
	--name nginx_SSL \		#容器名称
	-p 8011:80 -p 4433:443 \	#暴露端口 本机端口:容器端口
	--link blog_mini:blog_mini \	#关联blog_mini容器
	-v/data/nginx/config:/etc/nginx/conf.d \	#挂载本机的目录到容器
	-v /data/nginx/logs:/var/log/nginx \		#挂载本机的目录到容器
	nginx:1.13		#镜像
```
**查看所有的容器**
```bash
$ docker ps -a
CONTAINER ID    IMAGE           COMMAND     CREATED            STATUS                      PORTS      NAMES
8cae4be0d7cb    hello-world     "/hello"    32 seconds ago     Exited (0) 31 seconds ago              dreamy_noether
eccf37bf03b4    hello-world     "/hello"    40 minutes ago     Exited (0) 40 minutes ago              youthful_franklin
```
**常用Docker命令**
```bash
docker info			#显示Docker和系统信息
docker start hello-world	#启动容器
docker stop hello-world		#停止容器
docker exec -it centos /bin/bash	#执行centos容器的命令
docker inspect centos		#查看centos容器的信息
docker cp /root/xx.zip centos:/root	#复制文件到容器
docker rm hello-world		#删除hello-world容器
docker rmi hello-world		#删除hello-world镜像
```
