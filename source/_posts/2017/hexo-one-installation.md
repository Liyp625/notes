---
title: 利用Github Pages搭建Hexo个人博客网站
date: 2017-09-18 14:43:21
categories:
  - Blog
tags:
  - Hexo
  - GithubPages
---
之前自己搭建过一个博客是基于的xpleaf的[Blog_mini](https://github.com/xpleaf/Blog_mini)做了小优化的博客，那个博客是用Python+Flask做的，也蛮不错的，不过这个博客是需要购买云服务器的，得花money。最近突然想起来为何不弄一个免费的博客呢，这时Hexo进入我的视线，使用Hexo+Github Pages就可以免费搭建自己的个人博客网站。

<!-- more -->

### 1. 基础环境需要
Hexo是使用node.js构建的，所以基础环境如下：
- Node.js
- hexo
- Git

### 2. 安装Node.js和Git
我这使用的系统是Win，因此只要在官网上下载node.js和Git的安装包直接安装即可，很简单一切都是可视化的。
- [Node.js官网下载地址](https://nodejs.org/en/)
- [Git官网下载地址](https://git-scm.com/downloads)

对于CentOS系统的可以直接使用yum进行安装。
```bash
# 安装node.js
yum install -y nodejs 
# 安装git
yum install -y git
```
安装成功后可以使用node -v 和npm -v来检测node.js是否安装成功，使用it --version验证Git的安装是否成功
```bash
[root@VM_204_110_centos ~]# node -v
v6.11.3
[root@VM_204_110_centos ~]# npm -v
3.10.10
[root@VM_204_110_centos ~]# git --version
git version 1.8.3.1
```
### 3. 安装hexo
对于win系统，主要的操作都是使用git-bash，所以要打开git-bash程序，在里面执行相关的命令。
```bash
# 切换到要安装hexo的目录
$ cd /d/Documents/hexo/notes/
# 安装hexo
$ npm install hexo
```
*注：由于git-bash中的命令和CentOS系统中是一样，以下不再对CentOS做单独的说明。*

接下来对hexo进行初始化
```bash
# 初始化hexo
hexo init
# 安装hexo的相关依赖包
npm install
```
执行以上相关命令的时候需要联网下载，所以具体的话费时间需要看网络带宽了，执行成功后会生成如下目录
```bash
[root@VM_204_110_centos hexo]# ll notes/
total 32
-rw-r--r--   1 root root  1755 Sep 20 14:54 _config.yml
drwxr-xr-x 287 root root 12288 Sep 20 15:05 node_modules
-rw-r--r--   1 root root   443 Sep 20 14:54 package.json
drwxr-xr-x   2 root root  4096 Sep 20 14:54 scaffolds
drwxr-xr-x   3 root root  4096 Sep 20 14:54 source
drwxr-xr-x   3 root root  4096 Sep 20 14:54 themes
```
### 4. 创建新博文&本地验证Hexo
新建博文命令
```bash
hexo new "hi hexo"
```
这是会在source/_posts路径下生成一个名称为hi-hexo.md的Markdown文件，直接编辑该文件。而后执行如下命令。
```bash
# 清除以前生成的静态文件
hexo clean
# 生成静态文件，可以简写为hexo g
hexo generate
# 启动本地服务，可以简写为hexo s，默认端口是4000
hexo server
```
这时在浏览器的地址栏输入[http://localhost:4000](http://localhost:4000)即可看到自己的新建的hexo网站。

### 5. 将网站推送到Github上
既然要使用Github的Pages功能，Github的帐号是必须的了。
- [Github网址](https://github.com/)

#### 5.1. 创建repository
登录Github新建一个repository，注意名字必须要有一定的规则。比如你的用户名是`Hello`，那么这个repository的名字必须是`Hello.github.io`。
#### 5.2. 修改_config.yml配置文件
要把本地的静态网站推送到Github上必须要设置github的地址，在_config.yml中的Deployment位置修改。
```bash
deploy:
  type: git
  repository: https://github.com/Hello/Hello.github.io.git
  branch: master
```
*注：需要将Hello替换为自己实际的用户名*
#### 5.3. 部署
执行下列命令即可完成hexo的部署。

```bash
hexo clean
hexo generate
# 部署命令 可以简写为 hexo d
hexo deploy
```
直接访问地址[https://Hello.github.io](https://Hello.github.io)即可查看。
### 6. Hexo主题
Hexo支持很多主题，可以在Hexo官网上查找。
- [Hexo官网的主题](https://hexo.io/themes/)

同时在推荐一下[next主题](https://github.com/iissnan/hexo-theme-next)，具体的效果展示可以看作者的的博客[IIssNan's Notes](http://notes.iissnan.com)。