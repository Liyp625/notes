---
title: 修改npm的全局路径
date: 2020-11-25 15:03:12
categories:
  - 前端技术
tags:
  - Node.js
  - NPM
---
不想把所有的数据都放在系统盘，因此需要修改npm的全局路径

<!-- more -->

### 1. 查看npm的默认配置
```bash
npm config get
```
控制台输出信息
```bash
D:\Coder\nodejs>npm config get
; cli configs
metrics-registry = "https://registry.npmjs.org/"
scope = ""
user-agent = "npm/6.5.0-next.0 node/v11.6.0 win32 x64"

; builtin config undefined
prefix = "C:\\Users\\xxx\\AppData\\Roaming\\npm"

; node bin location = D:\Coder\nodejs\node.exe
; cwd = D:\Coder\nodejs
; HOME = C:\Users\xxx
; "npm config ls -l" to show all defaults.
```
*prefix* 为系统盘地址

### 2. 修改路径
1. 在要修改地址创建"node_global"及"node_cache"两个文件夹，用于后续的配置
2. 在命令行执行如下命令
```bash
# 全局安装位置
npm config set prefix "D:\Coder\nodejs\node_global"
# 缓存文件位置
npm config set cache "D:\Coder\nodejs\node_cache"
```
3. 查看、验证修改后的配置
```bash
npm config get
```
控制台输出信息
```bash
D:\Coder\nodejs>npm config get
; cli configs
metrics-registry = "https://registry.npmjs.org/"
scope = ""
user-agent = "npm/6.5.0-next.0 node/v11.6.0 win32 x64"

; userconfig C:\Users\Liyp\.npmrc
cache = "D:\\Coder\\nodejs\\node_cache"
prefix = "D:\\Coder\\nodejs\\node_global"

; builtin config undefined

; node bin location = D:\Coder\nodejs\node.exe
; cwd = D:\Coder\nodejs
; HOME = C:\Users\xxx
; "npm config ls -l" to show all defaults.
```
4. 修改环境变量（win）
需要把新路径（D:\Coder\nodejs\node_global）添加到环境变量

### 3. 恢复默认地址
```bash
# 全局安装位置
npm config delete cache
# 缓存文件位置
npm config delete prefix
```

### 4.使用taobao源加速npm
在要install或者update的命令后面附加 **--registry=https://registry.npm.taobao.org** 即可，如：
```bash
npm install xxx --registry=https://registry.npm.taobao.org
```