---
title: 'K8s'
description: 'k8s'
keywords: 'k8s'

date: 2024-03-13T22:25:52+08:00

categories:
  - k8s
tags:
  - k8s
  - docker
---

部署使用 k8s 平台。

<!--more-->

> 作者：WP&CZH\
> 出处：<https://znunwm.top/archives/k8s-xiang-xi-jiao-cheng>\
> 遵循 《署名-非商业性使用-相同方式共享 4.0 国际 (CC BY-NC-SA 4.0)》版权协议。

**本文参考以上文章进行部署和学习。**

## 介绍

### k8s 架构

![k8s-architecture](/imgs/posts/k8s/k8s-architecture.png)

Kubernetes 主要由以下几个核心组件组成：

- etcd 保存了整个集群的状态；
- kube-apiserver 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制；
- kube-controller-manager 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
- kube-scheduler 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上；
- kubelet 负责维持容器的生命周期，同时也负责 Volume（CVI）和网络（CNI）的管理；
- Container runtime 负责镜像管理以及 Pod 和容器的真正运行（CRI），默认的容器运行时为 Docker；
- kube-proxy 负责为 Service 提供 cluster 内部的服务发现和负载均衡；

![k8s-port](/imgs/posts/k8s/k8s-port.png)

除了核心组件，还有一些推荐的 Add-ons：

- kube-dns 负责为整个集群提供 DNS 服务
- Ingress Controller 为服务提供外网入口
- Heapster 提供资源监控
- Dashboard 提供 GUI
- Federation 提供跨可用区的集群
- Fluentd-elasticsearch 提供集群日志采集、存储与查询

## 搭建环境

三台虚拟机：

| hostname  | ip              | role   |
| --------- | --------------- | ------ |
| centos200 | 192.168.100.200 | master |
| centos201 | 192.168.100.201 | node01 |
| centos202 | 192.168.100.202 | node02 |

软件版本：

| 类别   | 版本                                 |
| ------ | ------------------------------------ |
| System | CentOS Linux release 7.9.2009 (Core) |
| Docker | 20.10                                |
| k8s    | 1.23.17                              |

### 设置主机静态 IP

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

修改内容如下：

```text
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
- BOOTPROTO=dhcp
+ BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=a33798c6-2dfc-4b33-b67d-5a8544f9a2c5
DEVICE=ens33
- ONBOOT=no
+ ONBOOT=yes
+ IPADDR=192.168.100.200
+ GATEWAY=192.168.100.1
+ DNS1=192.168.100.1
+ DNS2=223.5.5.5
```

### 设置主机名解析

为了方便集群节点直接调用，此处配置主机名称解析，生产环境中应使用内部 DNS 服务器。

```shell
vi /etc/hosts

192.168.100.200 centos200
192.168.100.201 centos201
192.168.100.202 centos202
192.168.100.203 centos203
192.168.100.204 centos204
192.168.100.205 centos205
192.168.100.206 centos206
192.168.100.207 centos207
192.168.100.208 centos208
192.168.100.209 centos209
```

### 关闭防火墙

关闭防火墙，避免后续端口不开放导致的各种连接原因。

```shell
systemctl stop firewalld && systemctl disable firewalld.service
```

### 设置 yum 源

此处为方便后续安装软件等服务，需要更新 yum 源，为方便安装，现提供相关命令如下：

```shell
cd /etc/yum.repos.d/
mkdir bak
mv *.repo bak/
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
yum install -y epel-release.noarch
yum clean all
yum makecache
yum repolist all
```

为后续可以直接安装 k8s 相关服务，此处也需要添加 k8s 相关源。

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 时间同步

k8s 要求集群中时间节点必须强一致，此处使用 chrony 服务来进行时间同步。

```shell
# 因为此处使用的是最小安装的 CentOS7 镜像，系统安装后并不带有此服务，因此需要提前进行安装
yum install –y chrony
# 设置开机自启动
systemctl start chronyd && systemctl enable chronyd
# 设置时区
timedatectl set-timezone "Asia/Shanghai" && timedatectl
# 配置阿里云的 ntp 服务
vi /etc/chrony.conf

- server 0.centos.pool.ntp.org iburst
- server 1.centos.pool.ntp.org iburst
- server 2.centos.pool.ntp.org iburst
- server 3.centos.pool.ntp.org iburst

+ server ntp1.aliyun.com iburst
+ server ntp2.aliyun.com iburst
+ server ntp3.aliyun.com iburst

# 重启服务，使其生效
systemctl start chronyd.service
systemctl enable chronyd.service
```

### 关闭 iptable

关闭系统 iptables 规则（后期 kubernetes 和 docker 在运行的中会产生大量的 iptables 规则，此处关闭系统规则，避免混淆）。

```shell
systemctl stop iptables && systemctl disable iptables
```

### 禁用 selinux

关闭 selinux 安全服务，避免集群安装过程中遇到的各种问题。

```shell
sed -i 's/enforcing/disabled/' /etc/selinux/config && setenforce 0
```

### 关闭 swap 分区

禁用 swap 分区，启用 swap 设备会对系统的性能产生非常负面的影响，因此 kubernetes 要求每个节点都要禁用 swap 设备

```shell
# 临时关闭
swapoff -a

# 永久关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

### 修改 linux 内核参数

增加网桥过滤和地址转发功能。

```shell
vi /etc/sysctl.d/kubernetes.conf
# 增加以下配置
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

# 重新加载配置
sysctl -p && modprobe br_netfilter
# 查看网桥模块是否加载成功
lsmod | grep br_netfilter
```

### 配置 ipvs

在 Kubernetes 中 Service 有两种带来模型，一种是基于 iptables 的，一种是基于 ipvs 的两者比较的话，ipvs 的性能明显要高一些，但是如果要使用它，需要手动载入 ipvs 模块。

```shell
# 安装 ipset 和 ipvsadm
yum install ipset ipvsadm -y
# 配置需要加载的模块
cat > /etc/sysconfig/modules/ipvs.modules << EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
# 授权执行加载
chmod +x /etc/sysconfig/modules/ipvs.modules
sh /etc/sysconfig/modules/ipvs.modules
# 查看对应模块加载情况
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

### 安装 docker

```shell
# 安装 wget
yum install -y wget
# 获取镜像源
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
# 查看可安装版本
yum list docker-ce --showduplicates | sort -r
# 安装指定版本的 docker
# 需要加上 --setopt=obsoletes=0，否则将会安装最新版本
yum install --setopt=obsoletes=0 docker-ce-20.10.9 -y
# 启动并设置开机自启动
systemctl start docker && systemctl enable docker
# 镜像加速
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ],
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

### 安装指定版本的 k8s

```shell
# 安装相关服务
yum install --setopt=obsoletes=0 -y kubelet-1.23.17 kubeadm-1.23.17 kubectl-1.23.17
# 配置kubelet的cgroup
vi /etc/sysconfig/kubelet
# 添加下面的配置
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"
# 设置开机自启
systemctl enable kubelet
```

### 复制出两台虚拟机，设置静态相应的静态 IP

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens33

# 修改 ip 行，分别设置为 201 和 202
- IPADDR=192.168.100.200
+ IPADDR=192.168.100.201
+ IPADDR=192.168.100.202

# 重启网络服务
service network restart
# 查看 ip
ip addr
```

### 初始化 master 节点

设置 centos200 为 master 节点

```shell
kubeadm init \
  --kubernetes-version 1.23.17 \
  --apiserver-advertise-address=0.0.0.0 \
  --service-cidr=10.96.0.0/16 \
  --pod-network-cidr=10.244.0.0/16 \
  --image-repository registry.aliyuncs.com/google_containers
```

按照指示执行下一步：

```text
To start using your cluster, you need to run the following as a regular user:
如果你使用的是普通用户，需要执行以下命令：

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:
如果你是使用的是 root 用户，需要执行以下命令：

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:
在子节点上执行以下命令：

kubeadm join 192.168.100.200:6443 --token d8vplm.t4ihd59gbmm0j8gd \
        --discovery-token-ca-cert-hash sha256:316e93f91360422fcc37f1e00676fabe466fc3e9dae38c6fa6e0a004f0e61c57
```

```shell
# 查看 token
kubeadm token list
# 生成新的 token
kubeadm token create --print-join-command
```

在 master 节点上查看节点信息：

```shell
kubectl get nodes
```

### 安装 kube-flannel

```shell
## master,node centos200.centos201,centos202 节点都需要执行
mkdir -p /etc/cni/net.d
cat > /etc/cni/net.d/10-flannel.conf << EOF
{
 "name":"cbr0",
 "type":"flannel",
 "deledate":{
    "hairpinMode":true,
    "isDefaultGateway":true
  }
}
EOF
mkdir -p /usr/share/oci-umount/oci-umount.d
mkdir -p /run/flannel
cat > /run/flannel/subnet.env << EOF
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1410
FLANNEL_IPMASQ=true
EOF
# master: centos200 节点执行
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

如果安装完成之后，长时间集群仍然是 noready，可使用 `journalctl -f -u kubelet.service` 查看原因。

### 同步 kubectl

```shell
# master: centos200
scp /etc/kubernetes/admin.conf root@centos201:/etc/kubernetes
scp /etc/kubernetes/admin.conf root@centos202:/etc/kubernetes
# node: centos201 centos202
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## 资源管理

ziyuanguanli
