---
title: Centos系列之一：Centos7安装VMwareTools
date: 2018-05-04 14:06:23
categories:
  - Linux
tags:
  - CentOS 7
  - VMware
---
在WMware上的Centos7虚拟机要与主机共享文件夹需要安装VMwareTools（通过命令行模式）。

<!-- more -->

### 1. 加载VMwareTools的镜像
首先启动CentOS 7,在VMware中点击上方“VM”，点击“Install VMware Tools...”（如已安装则显示“Reinstall VMware Tools...”）

*也可以直接使用虚拟机的光驱来手动加载VMware安装目录下的linux.iso镜像*
### 2. 在系统中挂载cdrom
```bash
# 创建一个用于挂载镜像的目录
mkdir /mnt/cdrom
# 使用mount命令来加载镜像到/mnt/cdrom目录下
mount /dev/cdrom /mnt/cdrom
```
使用ll命令来查看镜像的内容
```bash
[root@localhost ~]# ll /mnt/cdrom/
total 56377
-r-xr-xr-x. 1 root root     1975 Mar 17  2017 manifest.txt
-r-xr-xr-x. 1 root root     2287 Mar 17  2017 run_upgrader.sh
-r--r--r--. 1 root root 56072885 Mar 17  2017 VMwareTools-10.1.6-5214329.tar.gz
-r-xr-xr-x. 1 root root   802620 Mar 17  2017 vmware-tools-upgrader-32
-r-xr-xr-x. 1 root root   848816 Mar 17  2017 vmware-tools-upgrader-64
```
### 3. 解压WMwareTools.tar.gz
```bash
# 复制压缩包到home
cp /mnt/cdrom/VMwareTools-10.1.6-5214329.tar.gz ~
# 卸载镜像
umount /mnt/cdrom
# 解压VMwareTools到当前路径下
tar -zxcf ~/VMwareTools-10.1.6-5214329.tar.gz
```
解压完成后会在当前路径下生成一个vmware-tools-distrib的目录
```bash
[root@localhost ~]# ll
total 2764868
-rw-------. 1 root root       1238 May 10  2017 anaconda-ks.cfg
drwxr-xr-x. 3 root root        141 Oct 12  2017 CI
drwxr-xr-x. 4 root root         45 Nov 30 13:30 docker
-rw-r--r--. 1 root root 2457254895 May  3 16:32 harbor_db.tar.gz
-rw-r--r--. 1 root root  317888971 Aug 29  2017 harbor-offline-installer-0.5.0.tgz
-r--r--r--. 1 root root   56072885 May  4 08:54 VMwareTools-10.1.6-5214329.tar.gz
drwxr-xr-x. 9 root root        145 Mar 17  2017 vmware-tools-distrib
```
### 4. 安装Cenos7编译环境
输入“cd vmware-tools-distrib/”进入名为“vmware-tools-distrib”的目录，输入“./vmware-install.pl”尝试安装，出现错误“-bash: ./vmware-install.pl: /usr/bin/per: bad interpreter: No such file or directory”，表明未安装编译环境。
```bash
# 使用yum来安装编译环境
yum -y install perl gcc make kernel-headers kernel-devel
```
完成后使用yum命令来查看kernel(内核)的版本，需要保持一致
```bash
[root@localhost ~]# yum list kernel*
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Installed Packages
kernel.x86_64                                                      3.10.0-514.el7                                          @anaconda
kernel.x86_64                                                      3.10.0-514.16.1.el7                                     @updates
kernel.x86_64                                                      3.10.0-693.21.1.el7                                     @updates
kernel-devel.x86_64                                                3.10.0-693.21.1.el7                                     @updates
kernel-headers.x86_64                                              3.10.0-693.21.1.el7                                     @updates
kernel-tools.x86_64                                                3.10.0-693.21.1.el7                                     @updates
kernel-tools-libs.x86_64                                           3.10.0-693.21.1.el7                                     @updates
Available Packages
kernel-abi-whitelists.noarch                                       3.10.0-693.21.1.el7                                     updates
kernel-debug.x86_64                                                3.10.0-693.21.1.el7                                     updates
kernel-debug-devel.x86_64                                          3.10.0-693.21.1.el7                                     updates
kernel-doc.noarch                                                  3.10.0-693.21.1.el7                                     updates
kernel-tools-libs-devel.x86_64                                     3.10.0-693.21.1.el7                                     updates
```
如果内核不一致需要将内核升级到和kernel-headers、kernel-devel一致的版本
```bash
# 查看当前系统的内核
[root@localhost ~]# uname -a
Linux localhost 3.10.0-693.21.1.el7.x86_64 #1 SMP Wed Mar 7 19:03:37 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
### 5. 安装WMwareTools
进入vmware-tools-distrib目录
```bash
[root@localhost ~]# cd vmware-tools-distrib/
[root@localhost vmware-tools-distrib]# ll
total 368
drwxr-xr-x.  2 root root     87 Mar 17  2017 bin
drwxr-xr-x.  5 root root     39 Mar 17  2017 caf
drwxr-xr-x.  2 root root     67 Mar 17  2017 doc
drwxr-xr-x.  5 root root   4096 Mar 17  2017 etc
-rw-r--r--.  1 root root 149442 Mar 17  2017 FILES
-rw-r--r--.  1 root root   2538 Mar 17  2017 INSTALL
drwxr-xr-x.  2 root root    137 Mar 17  2017 installer
drwxr-xr-x. 15 root root    202 Mar 17  2017 lib
drwxr-xr-x.  3 root root     21 Mar 17  2017 vgauth
-rwxr-xr-x.  1 root root 216748 Mar 17  2017 vmware-install.pl
```
执行安装命令
```bash
./vmware-install.pl
```
安装过程中的人机交互可以直接全部回车
### 6. 安装完成后的提示
```bash
The configuration of VMware Tools 10.1.6 build-5214329 for Linux for this
running kernel completed successfully.

You must restart your X session before any mouse or graphics changes take
effect.

You can now run VMware Tools by invoking "/usr/bin/vmware-toolbox-cmd" from the
command line.

To enable advanced X features (e.g., guest resolution fit, drag and drop, and
file and text copy/paste), you will need to do one (or more) of the following:
1. Manually start /usr/bin/vmware-user
2. Log out and log back into your desktop session
3. Restart your X session.

Enjoy,

--the VMware team
```
### 7. 设置共享文件夹并挂载
在VMware软件的虚拟机设置来添加需要与虚拟机共享的主机文件夹。（VM -> Settings -> Options -> Shared folders）

在centos虚拟机上执行以下命令来挂载共享文件夹到/mnt/hgfs目录下
```bash
# 使用mount命令进行挂载
[root@localhost ~]# mount -t vmhgfs .host:/ /mnt/hgfs
# data1即为共享文件夹
[root@localhost ~]# ll /mnt/hgfs/
total 0
drwxrwxrwx. 1 root root 0 Nov 26 21:57 data
```