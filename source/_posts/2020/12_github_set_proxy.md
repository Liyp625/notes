---
title: GitHub设置网络代理
date: 2020-05-14 20:05:29
categories:
  - 开发工具
tags:
  - Git
  - GitHub
---
家里的网络有时候连接GitHub仓库很慢，可以通过设置GitHub的网络代理来解决

<!-- more -->

### 1. 查看Git的代理设置
```bash
git config --global -l
```
其中的http.proxy和https.proxy项即为Git的代理地址，如果这两项都没有则目前的git还没有设置代理
### 2. 设置代理
#### 2.1. HTTP代理
```bash
git config --global http.proxy http://127.0.0.1:808
git config --global https.proxy http://127.0.0.1:808
```
#### 2.2. SOCKS5代理
```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

### 3. 取消代理
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```
