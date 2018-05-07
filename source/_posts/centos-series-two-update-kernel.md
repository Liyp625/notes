---
title: Centos系列之二：Centos7升级内核
date: 2018-05-7 09:11:52
categories:
  - Linux
tags:
  - CentOS 7
---
centos7最简单的升级内核方法

<!-- more -->

### 1. 检查已安装的内核版本
```bash
# 使用uname命令查询当前的内核
[root@localhost ~]# uname -sr
Linux 3.10.0-693.21.1.el7.x86_64
```
### 2. 在 CentOS 7 中升级内核
使用最简单的yum命令来升级内核
```bash
yum update kernel
```
升级完成后重启系统，来使用新的内核
### 3. 设置 GRUB 默认的内核版本（可选）
为了让新安装的内核成为默认启动选项，需要如下修改 GRUB 配置：

打开并编辑`/etc/default/grub`并设置 `GRUB_DEFAULT=0`。意思是 GRUB 初始化页面的第一个内核将作为默认内核。
```bash
GRUB_TIMEOUT=5
GRUB_DEFAULT=0
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap crashkernel=auto rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```
接下来运行下面的命令来重新创建内核配置
```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```
重启并验证最新的内核已作为默认内核