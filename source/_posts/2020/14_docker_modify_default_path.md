---
title: 修改docker的默认存储路径
date: 2020-11-24 12:50:36
categories:
  - 容器
tags:
  - Docker
---
Docker默认安装的情况下，会以/var/lib/docker/ 作为默认存储目录，存放拉取的镜像和创建的容器等。以下说明如何修改Docker 的存储目录，以centos7为例

<!-- more -->

### 1. 查看docker的默认存储路径
```bash
# 查看docker的一些配置信息
docker info
```

控制台输出信息（部分）
```bash
Kernel Version: 3.10.0-862.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 1.787GiB
Name: master
ID: 235P:KF5C:FQSL:3KDM:Z5AB:63AJ:72XC:I5Y5:4B4E:CTA4:223Y:H6QM
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
```
*Docker Root Dir: /var/lib/docker* 即为docker的默认存储路径

### 2. 修改daemon.json文件
```bash
# 新建或编辑docker的daemon.json文件
vi /etc/docker/daemon.json
```
在配置文件中新增如下内容，/data/docker为要修改的地址
```bash
{ 
  "data-root": "/data/docker"
}
```
> docker的国内源也可以在这里进行修改，如添加163的源
```bash
{ 
  "data-root": "/data/docker",
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

### 3. 重启docker服务
```bash
systemctl restart docker
```