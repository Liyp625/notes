---
title: 常用Shell脚本命令
date: 2021-03-26 10:40:00
categories:
  - 脚本命令
tags:
  - shell
---

记录下日常经常使用到的Shell脚本的命令，以备不时之用，已包括sed

<!-- more -->

### 1. sed - 从日志中截取打印指定内容
需求：将kafka日志中的offset的数值提取打印出来，查看是否有中断

```bash
# a.log中的内容示例：xx offset: 500,par: 1
cat a.log | sed 's/,/\n/g' | grep "offset" | sed 's/:/\n/g' | sed '1d' | sed 's/ //g'
```
说明：
1. sed 's/,/\n/g'  将所有的逗号替换为换行符，
2. sed 's/:/\n/g'  将冒号替换为换行符；
3. sed '1d'  删除第1行
4. sed 's/ //g'  删除空格

### 2. wc 计算文件中的行数
```bash
wc -l cat a.log
```