---
title: 使用Github和TravisCI对Hexo持续集成
date: 2017-09-20 15:19:47
categories:
  - Blog
tags:
  - Travis CI
  - Github Pages
  - Hexo
---
自从使用Github Pages功能利用hexo创建了个人主页之后，就想利用持续集成来自动生成网页，于是著名的免费的持续集成网站Travis就进入了我的视野。

<!-- more -->

### 1. 持续集成的流程

使用hexo创建个人网站，在本地编辑文章完成后，提交推送到远程Github的master分支。由Travis检测到push后，自动从Github上拉取资源，执行hexo deploy命令，提交到Github的gh-pages分支，完成hexo个人网站的更新。

大致的流程如下图所示：

```
graph TD
A[本地修改文件]-->|push|B[Github的master分支]
B-->C[Travis监测到master分支的push]
C-->|编译之后push|D[Github的gh-pages分支]
```

### 2. Travis配置
#### 2.1. 登录Travis
直接在[Travis官网](https://www.travis-ci.org)中使用Github的帐号进行登录。

#### 2.2. 项目开启TravisCI
在Github的项目列表中选择开启hexo的项目。
![项目开启TravisCI](https://wx4.sinaimg.cn/mw690/4ca4c33cly1fl8bom7uogj20sb0j8q9c.jpg "项目开启TravisCI")
在项目的设置中开启`Build only if .travis.yml is  present`这一项，当travis.yml配置文件存在的时候才进行自动的持续集成。

#### 2.3. 在Github中生成Access Token

在Github的Settings -> Developer settings -> Personal access tokens中新增一个token，在生成这个token时注意要保存这个token（只显示一次）。

#### 2.4. 安装Travis
> 注意：需要安装Ruby环境

```bash
# 安装travis
gem install travis

# 在项目路径生成配置文件
touch .travis.yml

# 登录travis，需要输入github的帐号和密码
travis login

# 生成koten。REPO_TOKEN为变量名，TOKEN为Github生成的token
travis encrypt 'REPO_TOKEN=<TOKEN>' --add
```
之后会在.trvis.yml文件中新增如下内容，把这些内容添加到后面的Travis配置文件中
```bash
env:
  global:
    secure: bUuBpthJl9M9ybmpI0vkwHURwmh...
```

#### 2.5. 新增Travis的配置文件
在hexo项目的根目录下新增.travis.yml配置文件，具体内容如下：
```bash
language: node_js
node_js:
  - "6" # nodejs的版本
cache:
  directories:
    - node_modules
branches:
  only:
    - master # 设置自动化部署的源码分支
env:
  global:
    secure: bUuBpthJl9M9ybmpI0vkwHURwmh...
before_install:
  - export TZ='Asia/Shanghai' # 设置时区
  - npm install -g hexo-cli
install:
  - npm install  # 安装依赖组件
  - npm install hexo-generator-searchdb --save
  - npm install hexo-deployer-git --save
before_script:
  # 设置github账户信息
  - git config --global user.name "userName" #设置github用户名
  - git config --global user.email "userEmail" #设置github用户邮箱
  # github仓库操作_替换hexo的_config.yml配置文件中的原github地址，使用https+token来提交
  # git@github.com:userName/notes.git为原地址
  # https://${REPO_TOKEN}:x-oauth-basic@github.com/userName/notes.git为替换后的新地址
  # 其中${REPO_TOKEN}为生成token的变量名
  - sed -i'' "s~git@github.com:userName/notes.git~https://${REPO_TOKEN}:x-oauth-basic@github.com/userName/notes.git~" _config.yml
# 执行的命令
script:
  - hexo clean
  - hexo generate
# 执行的成功后执行
after_success:
  - hexo deploy
```

### 3. 提交新博文
在本地编辑新的博文，提交md文件至Github的远程仓库，然后可以在[Travis官网](https://travis-ci.org)查看项目自动化执行的过程和结果。