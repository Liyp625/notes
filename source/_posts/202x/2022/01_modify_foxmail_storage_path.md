---
title: 修改Foxmail的邮件存储位置
date: 2022-08-29 09:19:00 
categories:
  - 日常软件
tags:
  - Foxmail
  - Windows
---

Foxmail没有提供可视化界面的方式来修改邮件的存储位置，但是可以通过修改配置文件来实现邮件存储位置的变更。

<!-- more -->

### 1. 找到配置文件

Foxmail存储邮件的路径在名字为FMStorage.list的配置文件中

```bat
rem 配置文件位置示例
C:\Program Files\Foxmail 7.2\FMStorage.list

rem 配置文件中的内容如下
Storage\xxx@163.com\
Storage\yyy@126.com\
```

### 2. 迁移数据
Storage默认在Foxmail的安装目录下，如：
C:\Program Files\Foxmail 7.2\Storage

先关闭Foxmail软件，将安装目录下Storage文件夹里面的所有内容剪切到想要转移的目录下，例如：D:\Documents\Foxmail\Storage

### 3. 修改配置文件
将FMStorage.list中的配置按照最新的路径进行修改，如

```bat
rem FMStorage.list文件中的内容：
D:\Documents\Foxmail\Storage\xxx@163.com\
D:\Documents\Foxmail\Storage\yyy@126.com\
```

之后启动软件即可。