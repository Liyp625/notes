---
title: Docker系列之二：构建CentOS7
date: 2017-08-24 08:07:05
categories:
  - 容器
tags:
  - Docker
---
CentOS7的docker默认情况下是不能够执行systemctl命令的，抛出以下错误:
`Failed to get D-Bus connection: Operation not permitted`
从CentOS的官网上可以查询到解决该问题的办法，使用Dockerfile的方式来构建centos7的image。

<!-- more -->

### 1. CentOS7的Dockerfile
Dockerfile的具体内容如下：
```bash
# centos的版本
FROM centos:7
ENV container docker
# 在新建的images中使用systemctl命令
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*; \
# 安装ssh软件包（可选）
yum -y install openssh-server; \
yum -y install openssh-clients; \
# 安装net工具（可选）
yum -y install net-tools; \
yum clean all;
VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]
```
执行命令如下命令即可完成centos7的image的构建
```bash
$ docker build --rm -t centos7:base .
```
### 2. 运行CentOS7的Container
在构建完成之后，执行以下命令，即可运行
```bash
$ docker run --name centos_test -v /sys/fs/cgroup:/sys/fs/cgroup:ro --privileged -itd centos7:base
```
进入centos_test容器内部
```bash
$ docker exec -it test bash
```
启动ssh，不再报错，构建成功。
```bash
$ systemctl start sshd
```