---
title: 使用putty批量执行远程主机命令的bat脚本
date: 2021-03-25 09:39:00
categories:
  - bat脚本
tags:
  - bat
  - putty
  - Linux
  - 脚本
---

在 window 系统中，利用 Putty 工具实现 ssh 登录连接到远程的 Linux 系统上，可以实现 linux 中的远程 ssh、远程 scp 命令的功能

<!-- more -->

### 1. 安装 Putty

在[Putty 下载页面](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)中，根据自己客户端的实际情况选择相应的版本，之后进行安装。

### 2. 编写 Bat 脚本

以下为之前已经使用过的脚本，主要的功能如下：

- svn 更新代码
- maven 编译打包
- 以日期格式修改 jar 包的名字
- 上传 jar 包到远程 linux 服务器上
- 执行远程 linux 服务器上的 shell 脚本，启动服务

具体的逻辑可以看脚本中的注释，其中使用到了 Putty 中的 plink（实现 ssh 功能）、pscp（实现 scp 功能）。

```bat
@echo off

set inputStr=%1%
set isUpload=%2%

echo "输入参数：%inputStr%"
if "swdj"=="%inputStr%" (
	set jarName=zzs_swdj
	set jarPath=D:\Developer\IdeaProjects\swdj\target\zzs_swdj.jar
	set projectDir=D:\Developer\IdeaProjects\dzfp_xc\zzs_swdjkphd
	rem 远程服务器信息
	set remoteBashPath=/data/swdj_startup.sh
) ^
else if "fpkj"=="%inputStr%" (
	set jarName=zzs_fpkj
	set projectDir=D:\Developer\IdeaProjects\fpkj
	set jarPath=D:\Developer\IdeaProjects\fpkj\target\zzs_fpkj.jar
	rem 远程服务器信息
	set remoteBashPath=/data/fpkj_startup.sh
) ^
else (
	echo "参数错误"
)
echo %jarName%

set jarDir=D:\Developer\IdeaProjects\

rem 远程服务器信息 #性能：124.70.81.5 功能：119.3.224.85
set remoteIp=124.70.81.5
set remoteUser=root
set remotePwd=LWiQ1U0B
set remoteDirPath=/data/jars/%jarName%
rem set remoteBashPath=/data/fpkj_startup.sh

if "true" == "%isUpload%" (
	echo ">>>>>>>>>>>>>>>>>>>只是上传文件"
	goto :upload
)

rem >>>>>>>>>>>>>>>>>>>>>> svn更新
echo ">>> 开始更新应用"
echo %projectDir%
cd /d %projectDir%
svn update

rem >>>>>>>>>>>>>>>>>>>>>> maven构建
echo ">>> 开始构建应用"
call mvn clean package
echo ">>> 构建应用成功"

rem >>>>>>>>>>>>>>>>>>>>>> 复制jar包
echo ">>> 复制jar包"
cd /d %jarDir%

echo "%date:~0,4%%date:~5,2%%date:~8,2%"
set dateStr="%date:~5,2%%date:~8,2%"
set timeStr="%time:~0,2%%time:~3,2%"

set /a jarNum=0
for /f %%i in ('dir /b/o-d/a-d %jarDir%\%jarName%_%dateStr%*.jar') do (
	set /a jarNum += 1
)
set /a jarNum += 1
set newJarName=
if %jarNum% lss 10 (
	set newJarName=%jarName%_%dateStr%_0%jarNum%_dev128.jar
) else (
	set newJarName=%jarName%_%dateStr%_%jarNum%_dev128.jar
)
copy %jarPath% %newJarName%
echo ">>> 复制jar包成功"
echo %newJarName%

rem >>>>>>>>>>>>>>>>>>>>>> 上传jar包
goto :ok
:upload

echo "远程服务器IP：%remoteIp%"
echo %remoteDirPath%
for /f %%i in ('dir /b/o-d/a-d %jarDir%\%jarName%*.jar') do (
	echo %%i
	echo ">>> 开始上传文件"
	pscp -l %remoteUser% -pw %remotePwd% -P 22 -r ./%%i %remoteUser%@%remoteIp%:%remoteDirPath%
	echo ">>> 上传文件成功"

	echo ">>> 执行远程脚本"
	plink -batch -pw %remotePwd% -P 22 %remoteUser%@%remoteIp% "cd /data"
	plink -batch -pw %remotePwd% -P 22 %remoteUser%@%remoteIp% %remoteBashPath%

	echo ">>> 执行远程命令"
	TIMEOUT /T 3
	plink -batch -pw %remotePwd% -P 22 %remoteUser%@%remoteIp% "ls -l /data"
	echo ">>> 查询端口号，请等候..."
	TIMEOUT /T 10
	plink -batch -pw %remotePwd% -P 22 %remoteUser%@%remoteIp% "netstat -nplt"

	goto :ok
)

:ok
pause
```

### 3. 执行 Bat 脚本

bat 脚本有两个参数：

1. jar 包或者项目的名称，if 分支处理
2. 是否只是上传 jar 包，默认是只编译打包，不上传

示例 1：编译打包 swdj 的项目

```bat
build_zspringboot.bat swdj
```

示例 2：上传 swdj 的 jar 包到远程 linux 服务器上

```bat
build_zspringboot.bat swdj true
```
