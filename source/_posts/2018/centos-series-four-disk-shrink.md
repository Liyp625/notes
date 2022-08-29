---
title: Centos系列之四：压缩VMware的centos7虚机在宿主机上的磁盘空间
date: 2018-08-15 10:21:39
categories:
  - Linux
tags:
  - CentOS
  - VMware
---
随着VMware虚拟机的使用，centos在宿主机上的磁盘占用也会越来越大，有时候即使删掉了centos的多余文件也不会感觉到磁盘占用在减少

<!-- more -->

### 1. 使用shrink命令压缩磁盘空间
在VMware上的虚拟机系统要想压缩实际的磁盘空间的占用，前提是必须安装了VMwareTools或者open-vm-tools

下面以centos7的open-vm-tools为例

使用 *df* 命令可以查看虚机目前的磁盘占用，实际上要小于宿主机的磁盘占用空间。
```bash
[root@localhost ~]# df -h
```
使用**shrink**来压缩磁盘空间
```
[root@localhost ~]# vmware-toolbox-cmd disk shrink /
Please disregard any warnings about disk space for the duration of shrink process.
Progress: 100 [===========>]
```
在下面的进度条到了100的时候，这时的宿主机的VMware软件也会弹出一个含有进度条的提示框，**提示“正在压缩磁盘XXX.vmdk”**，当这个进度条走完后，压缩就成功了，之后就可以查看下虚机文件夹的磁盘空间大小。