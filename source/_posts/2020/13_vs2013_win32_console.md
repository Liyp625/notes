---
title: VS2013在调试win32程序时附加显示Console窗口
date: 2020-10-13 10:20:18
categories:
  - C++
tags:
  - VS2013
  - C++
---
使用VS2013在调试win32程序，不能显示Console窗口，部分调试信息无法显示，非常不便，可以通过vs的设置显示console窗口

<!-- more -->

### 1. VS2013设置
1. vs中打开解决方案。
2. 鼠标移动到项目名称上，点击鼠标右键，再点击属性，此刻会此项目的属性页。
3. 在配置属性->生成事件->后期生成事件中添加以下内容
```bash
editbin /SUBSYSTEM:CONSOLE $(OUTDIR)\$(ProjectName).exe
```
4. 运行程序时多弹出一个黑色命令行的提示框，用来输出调试信息