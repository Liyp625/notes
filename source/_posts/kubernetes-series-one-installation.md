---
title: Kubernetes系列之一：安装步骤
date: 2017-09-04 09:30:11
categories:
  - 容器
tags:
  - Kubernetes
  - Dokcer
---
Docker是一个很好的东西，特别是在和微服务相结合之后。但是在服务增多，同时需要增多服务节点时，过多的Docker容器的管理就变得很麻烦。Kubernetes就是一款Google发布的开源的容器集群管理软件，它非常利于对Docker容器集群做到自动化部署、管理，能弹性地调整容器的个数、扩展性能，自动维护容器。本文主要介绍Kubernetes的安装步骤以及部分常用插件的安装。

<!-- more -->

### 1. Kubernetes安装
以下是两个节点的部署示例，下表为各节点的IP信息。

| 节点 | 主机名 | IP地址 |
| ------------ | ------------ | ------------ |
| master | k8s-master | 192.168.12.210  |
| node1 | k8s-node1 | 192.168.12.211 |

`注意：需要所有节点都应该关闭防火墙`
```bash
systemctl disable firewalld #禁止系统启动时服务自启动
systemctl stop firewalld #停止服务
```

系统/软件版本

| 系统/软件 | 版本 |
| ------------ | ------------ |
| centos | 7.3 |
| docker | 1.12.6 |
| kubernetes | 1.5.2 |
#### 1.1. master节点安装
##### 1.1.1. 修改master的主机信息
首先修改master节点的hostname为k8s-master
```bash
hostnamectl set-hostname k8s-master
```
添加hosts文件的内容
```bash
echo "k8s-master 192.168.12.210" >> /etc/hosts
echo "k8s-node1 192.168.12.211" >> /etc/hosts
```
##### 1.1.2. 安装kubernetes、etcd、flannel
使用yum来安装kurbernets
```bash
yum install -y kubernetes etcd flannel
```
##### 1.1.3. 修改kubernetes的配置文件
查看/etc/kubernetes下的配置文件
```bash
ll /etc/kubernetes/
total 24
-rw-r--r--. 1 root root 1028 Sep  1 04:47 apiserver
-rw-r--r--. 1 root root  656 Aug 30 03:27 config
-rw-r--r--. 1 root root  189 Mar  6  2017 controller-manager
-rw-r--r--. 1 root root  768 Aug 31 01:15 kubelet
-rw-r--r--. 1 root root  103 Mar  6  2017 proxy
-rw-r--r--. 1 root root  111 Mar  6  2017 scheduler
```
修改apiserver
```bash
# The address on the local server to listen to.
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"
# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
# default admission control policies
# 去除了ServiceAccount--需要认证
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
# Add your own!
KUBE_API_ARGS=""
```
修改config
```bash
KUBE_LOGTOSTDERR="--logtostderr=true"
# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"
# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"
# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://k8s-master:8080"
```
修改kubelet
```bash
# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=0.0.0.0"
# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=k8s-master"
# location of the api-server
KUBELET_API_SERVER="--api-servers=http://k8s-master:8080"
# pod infrastructure container
# 这个地址是使用的私有的harbor地址（docker images的仓库）
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=192.168.12.200/myproj/pause-amd64:3.0"
```
##### 1.1.4. 配置etcd
```bash
vi /etc/etcd/etcd.conf
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"
```
##### 1.1.5. 注册&配置FLANNEL
```bash
etcdctl mkdir /kube-centos/network
etcdctl mk /kube-centos/network/config "{ \"Network\": \"172.30.0.0/16\", \"SubnetLen\": 24, \"Backend\": { \"Type\": \"vxlan\" } }"
```
```bash
vi /etc/sysconfig/flanneld
FLANNEL_ETCD="http://k8s-master:2379"
FLANNEL_ETCD_KEY="/kube-centos/network"
FLANNEL_OPTIONS=""
```
##### 1.1.6. 重启服务
```bash
for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler flanneld; do
	systemctl restart $SERVICES
	systemctl enable $SERVICES
	systemctl status $SERVICES
done
```
#### 1.2. node节点安装
##### 1.2.1. 修改node1的主机信息
首先修改node1节点的hostname为k8s-node1
```bash
hostnamectl set-hostname k8s-node1
```
添加hosts文件的内容
```bash
echo "k8s-master 192.168.12.210
k8s-node1 192.168.12.211" >> /etc/hosts
```
##### 1.2.2. 安装kubernetes和flannel
使用yum来安装kurbernets
```bash
yum install -y kubernetes-node flannel
```
##### 1.2.3. 修改kubernetes的配置文件
查看/etc/kubernetes下的配置文件
```bash
ll /etc/kubernetes/
total 12
-rw-r--r--. 1 root root 655 Aug 30 03:41 config
-rw-r--r--. 1 root root 769 Aug 30 22:59 kubelet
-rw-r--r--. 1 root root 103 Mar  6  2017 proxy
```
修改config文件如下
```bash
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"
# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"
# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=true"
# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://k8s-master:8080"
```
修改kubelet文件如下
```bash
# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=0.0.0.0"
# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=k8s-slave1"
# location of the api-server
KUBELET_API_SERVER="--api-servers=http://k8s-master:8080"
# pod infrastructure container
# 这个地址是使用的私有的harbor地址（docker images的仓库）
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=192.168.12.200/myproj/pause-amd64:3.0"
```
##### 1.2.4. 修改flannel的配置文件
```bash
vi /etc/sysconfig/flanneld
```
```bash
# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD_ENDPOINTS="http://k8s-master:2379"
# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
# 必须与master节点配置flannel时的路径一致
FLANNEL_ETCD_PREFIX="/kube-centos/network"
# Any additional options that you want to pass
#FLANNEL_OPTIONS=""
```
##### 1.2.5. 重启服务
```bash
for SERVICES in kube-proxy kubelet flanneld docker; do
	systemctl restart $SERVICES
	systemctl enable $SERVICES
	systemctl status $SERVICES
done
```
此时在master节点执行如下命令，验证是否能够查到该节点
```bash
$ kubectl get nodes
NAME        STATUS    AGE
k8s-node1   Ready     6d
```
### 2. skydns安装
Kubernetes的DNS插件
1) 先下载好四个组件包
`harbor.xxx.com/kube/kubedns-amd64:1.9`
`harbor.xxx.com/kube/kube-dnsmasq-amd64:1.4`
`harbor.xxx.com/kube/dnsmasq-metrics-amd64:1.0`
`harbor.xxx.com/kube/exechealthz-amd64:1.2`
2) 修改skydns-rc.yaml中对应的images地址
3) 安装资源
```bash
kubectl create -f skydns-rc.yaml
kubectl create -f skydns-svc.yaml
```
4) 在所有节点的/etc/kubernetes/kubelet最后一行修改为：
```bash
KUBELET_ARGS="--cluster-dns=10.254.0.100 --cluster-domain=cluster.local"
```
### 3. dashboard安装
Kubernetes的WebUI
1) 创建、下载dashboard的镜像
`harbor.xxx.com/kube/k8s‐dashboard:1.6.0`
2) 使用kubenetes安装文件中的yaml文件来创建资源
，修改如下字段
```bash
image: harbor.xxx.com/kube/k8s-dashboard:1.6.0
imagePullPolicy: IfNotPresent
 - --apiserver-host=http://192.168.12.210:8080
```
3) 安装资源
```bash
kubectl create -f kubernetes-dashboard.yaml
```
### 4. heapster安装
Kubernetes的性能监控插件
1) 下载组件包：
`harbor.xxx.com/kube/heapster:canary`
`harbor.xxx.com/kube/heapster_influxdb:v0.5`
`harbor.xxx.com/kube/heapster_grafana:v2.6.0-2`
2) 修改heapster的配置文件，文件位置解压heapster-1.2.0.tar.gz文件
`heapster-1.2.0/deploy/kube-config/influxdb/`
3) 修改heapster-controller.yaml文件中对应images，然后修改最后两句为
```bash
 - --source=kubernetes:http://192.168.12.210:8080?inClusterConfig=false
 - --sink=influxdb:http://monitoring-influxdb:8086
```
4) 修改influxdb-grafana-controller.yaml文件中的对应images地址。
5) 安装资源
```bash
# (在heapster-1.2.0/deploy/kube-config/influxdb/目录中执行)
kubectl create -f .
```