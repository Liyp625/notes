---
title: 使用shell脚本通过ssh批量执行命令
date: 2021-03-26 10:40:00
categories:
  - 脚本命令
tags:
  - Shell
  - Linux
---

通过提供 ip 地址，在多个远程 linux 主机上执行的 bash 命令，包括 ssh、scp

<!-- more -->

### 1. shell 脚本说明

shell 脚本类型:

- ssh：远程执行 linux 命令
- cp：复制文件到远程服务器
- ssh_local：在远程服务器上，执行本地的 shell 脚本的命令

### 2. shell 脚本内容

> 该脚本连接远程 linux 是通过免密登录的方式，用户名密码登录可以借助sshpass命令完成
```bash
sshpass -p 密码 ssh root@localhost
sshpass -p 密码 scp ~/file root@localhost:/tmp/file
```

#### 2.1. 主脚本内容

cmd-batch.sh

```bash
#!/bin/bash
#echo "TYPE: $1"

user_name="root"
#pem_path="-i ./.ssh/ecs.pem"
pem_path=""
timeout=3

echo "_______________________________________________________________________________"

if [[ $1 = "ssh" ]];then
  echo ">>>>>>>>>>>>>>>>>>>> IP: $2 -> CMD: $3 <<<<<<<<<<<<<<<<<<<<"
  ssh -o "StrictHostKeyChecking no"  -o ConnectTimeout=${timeout} ${pem_path}  ${user_name}@$2 $3
  if [[ $? -ne 0 ]];then
    echo "command exec failed"
  fi
  #result=`ssh -o "StrictHostKeyChecking no"  -o ConnectTimeout=${timeout} -i ${pem_path}  ${user_name}@$2 $3`
  #echo ${result}
elif [[ $1 = "cp" ]];then
  echo ">>>>>>>>>> IP: $2 -> localPath: $3 -> remotePath: $4 <<<<<<<<<<"
  scp -o "StrictHostKeyChecking no"  -o ConnectTimeout=${timeout} ${pem_path} $3 ${user_name}@$2:$4
  if [[ $? -ne 0 ]];then
    echo "command exec failed"
  fi
elif [[ $1 = "ssh_local" ]];then
  echo ">>>>>>>>>>>>>>>>>>>> IP: $2 -> CMD: $3 <<<<<<<<<<<<<<<<<<<<"
  ssh -o "StrictHostKeyChecking no"  -o ConnectTimeout=${timeout} ${pem_path}  ${user_name}@$2 < $3
  if [[ $? -ne 0 ]];then
    echo "command exec failed"
  fi
fi
```

#### 2.2. ip 地址文件

ecs-ip-list.txt

```
192.168.100.1
192.168.100.2
192.168.100.3
```

#### 2.3. 批量复制文件脚本

cp-batch.sh

```bash
#!/bin/bash
for ip in `cat ./ecs-ip-list.txt`
do
  #echo $line;
  #ip=$line;
  if [[ ${ip} != *#* ]]; then
  ./cmd-batch.sh "cp" ${ip} "$1" "$2"
  fi
done
```

#### 2.4. 批量执行命令脚本

ssh-batch.sh

```bash
#!/bin/bash
for ip in `cat ./ecs-ip-list.txt`
do
  if [[ ${ip} != *#* ]]; then
  ./cmd-batch.sh "ssh" ${ip} "$1"
  fi
done
```

#### 2.5. 批量执行本地命令脚本

ssh-local-batch.sh

```bash
#!/bin/bash
for ip in `cat ./ecs-ip-list.txt`
do
  if [[ ${ip} != *#* ]]; then
  ./cmd-batch.sh "ssh_local" ${ip} "$1"
  fi
done
```

#### 2.6. 单个执行命令脚本

ssh-single.sh

```bash
echo "IP: $1"
./cmd-batch.sh "ssh" $1 "$2"
```

### 3. shell 脚本执行示例

示例 1：

```bash
./ssh-batch.sh ll
```

示例 2：

```bash
./cp-batch.sh "a.jar" "/data/jars/"
```
