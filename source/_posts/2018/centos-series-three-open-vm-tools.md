---
title: Centos系列之三：Centos7安装open-vm-tools和设置共享文件夹
date: 2018-08-14 16:34:31
categories:
  - Linux
tags:
  - CentOS 7
  - VMware
---
最近在给VMware的CentOS7的虚拟机安装VMwareTools的时候，发现有一个提示说是现在open-vm-tools已经替代了VMwareTools。

<!-- more -->

### 1. 什么是open-vm-tools
open-vm-tools 是 VMware Tools 的开源实施，由一套虚拟化实用程序组成，这些程序可增强虚拟机在 VMware 环境中的功能，使管理更加有效。open-vm-tools 的主要目的是使操作系统供应商及/或社区以及虚拟设备供应商将 VMware Tools 绑定到其产品发布中。

系统版本要求
- Red Hat Enterprise Linux 7.0及以上
- SUSE Linux Enterprise 12及以上
- Ubuntu 14.04及以上
- CentOS 7 and及以上
- Debian 7.x及以上
- Oracle Linux 7及以上
- Fedora 19及以上
- openSUSE 11.x及以上

> 源码地址[：vmware/open-vm-tools](https://github.com/vmware/open-vm-tools)

### 2. 安装open-vm-tools

centos安装比较简单直接使用yum安装即可
```bash
yum install open-vm-tools
```
验证是否安装成功，输入命令vmware-直接按tab键，如果自动出现下面的这些命令就表示已经安装成功了
```bash
vmware-checkvm             vmware-hgfsclient          vmware-rpctool             vmware-vgauth-cmd          
vmware-guestproxycerttool  vmware-namespace-cmd       vmware-toolbox-cmd         vmware-xferlogs            
```
### 3. 共享宿主机的文件夹
在VMware软件的虚拟机设置来添加需要与虚拟机共享的主机文件夹。（VM -> Settings -> Options -> Shared folders）

在centos虚拟机上执行以下命令来挂载共享文件夹到/mnt/hgfs目录下
```bash
# 挂载所有的共享文件夹
vmhgfs-fuse .host:/ /mnt/hgfs
# 只挂载/foo/bar文件夹
vmhgfs-fuse .host:/foo/bar /mnt/hgfs
```
验证是否成功
```bash
ll /mnt/hgfs/
total 0
drwxrwxrwx. 1 root root 0 Aug 16 16:41 data
```
data文件夹就是之前设置的要共享的宿主机的文件夹

但是上面的操作只是临时的，重新启动系统后，共享文件夹就会失效，想要永久生效还需要进行其他的配置
```bash
# 修改配置
vi /etc/rc.local
```
在文件的下面新增以下内容
```bash
# mount shared floder
vmhgfs-fuse .host:/ /mnt/hgfs
```
/etc/rc.local其实是/etc/rc.d/rc.local的一个软连接，需要保证/etc/rc.d/rc.local具有可执行权限
```bash
# 新增执行权限
chmod +x /etc/rc.d/rc.local
```
至此open-vm-tools的安装和共享文件夹的设置已经全部完成，可以放心大胆地使用了