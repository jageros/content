---
type: "posts"
author: "jager"
title: "Kubernetes服务部署步骤"
date: "2021-07-30"
tags: [
"go",
"k8s",
]
---

> kubernetes，简称K8s，是用8代替8个字符“ubernete”而成的缩写。
> 是一个开源的，用于管理云平台中多个主机上的容器化的应用，
> Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,
> Kubernetes提供了应用部署，规划，更新，维护的一种机制。
> 本文将介绍Kubernetes集群的部署步骤

<!--more-->

## 一、环境要求

+ 物理机 × 3 (master × 1 + node × 2)【4核8G】
+ Linux发行版：CentOS7
+ Linux内核：3.10.0-1160.25.1.el7.x86_64
+ docker-ce 18.09.9
+ k8s v1.16.0

## 二、Kubernetes集群搭建

### 1、修改机器参数

```shell
# 分别修改三台机器的主机名
hostnamectl set-hostname master
hostnamectl set-hostname node1
hostnamectl set-hostname node2
```

### 2、安装docker（所有机器）

```shell
# 安装docker所需的工具
yum install -y yum-utils device-mapper-persistent-data lvm2
# 配置阿里云的docker源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 指定安装这个版本的docker-ce
yum install -y docker-ce-18.09.9-3.el7
# 启动docker
systemctl enable docker && systemctl start docker
```

### 3、设置k8s环境变量（所有机器）

```shell
# 关闭防火墙
systemctl disable firewalld
systemctl stop firewalld
 
# 关闭selinux
# 临时禁用selinux
setenforce 0
# 永久关闭 修改/etc/sysconfig/selinux文件设置
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
 
# 禁用交换分区
swapoff -a
# 永久禁用，打开/etc/fstab注释掉swap那一行。
sed -i 's/.*swap.*/#&/' /etc/fstab
 
# 修改内核参数
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 4、安装配置master节点（只在master操作）

```shell
# 安装kubeadm、kubelet、kubectl
# 由于官方k8s源在google，国内无法访问，这里使用阿里云yum源
# 执行配置k8s阿里云源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
 
# 安装kubeadm、kubectl、kubelet
yum install -y kubectl-1.16.0-0 kubeadm-1.16.0-0 kubelet-1.16.0-0
 
# 启动kubelet服务
systemctl enable kubelet && systemctl start kubelet
 
# 初始化k8s 以下这个命令开始安装k8s需要用到的docker镜像
# 因为无法访问到国外网站，所以这条命令使用的是国内的阿里云的源(registry.aliyuncs.com/google_containers)。
# 另一个非常重要的是：这里的--apiserver-advertise-address使用的是master和node间能互相ping通的master ip，测试环境中是192.168.99.104，请修改成自己的ip再执行。
# 这条命令执行时会卡在[preflight] You can also perform this action in beforehand using ''kubeadm config images pull，大概需要2分钟，请耐心等待。

kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.16.0 --apiserver-advertise-address 192.168.60.143 --pod-network-cidr=10.244.0.0/16 --token-ttl 0
 
# 上面安装完成后，k8s会提示你输入如下命令，执行
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
# 查询node 加入 master的命令
kubeadm token create --print-join-command

```

### 5、安装node工作节点（所有node节点）

```
# 安装kubeadm、kubelet
# 执行配置k8s阿里云源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
 
# 安装kubeadm、kubectl、kubelet
yum install -y  kubeadm-1.16.0-0 kubelet-1.16.0-0
 
# 启动kubelet服务
systemctl enable kubelet && systemctl start kubelet
 
# 加入集群，如果这里不知道加入集群的命令，可以登录master节点，使用kubeadm token create --print-join-command 来获取
kubeadm join 192.168.99.104:6443 --token ncfrid.7ap0xiseuf97gikl \
    --discovery-token-ca-cert-hash sha256:47783e9851a1a517647f1986225f104e81dbfd8fb256ae55ef6d68ce9334c6a2

```

### 6、安装flannel网络组件（只在master操作）

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 安装完成之后，可通过 kubectl get nodes 命令查看所有节点是否都为Ready状态，全为Ready说明节点间网络已打通了
```

### 7、部署Ingress控制器：traefik v2.3

+ traefik介绍

  > Traefik 是一个边缘路由器。由于 Traefik 2.X 版本和之前的 1.X 版本不兼容，我们这里选择功能更加强大的 2.X 版本来和大家进行讲解，我们这里使用的镜像是 `traefik:2.3`。
  >
  > 在 Traefik 中的配置可以使用两种不同的方式：
  >
  > - 动态配置：完全动态的路由配置
  > - 静态配置：启动配置
  >
  > `静态配置`中的元素（这些元素不会经常更改）连接到 providers 并定义 Treafik 将要监听的 entrypoints。
  >
  > > 在 Traefik 中有三种方式定义静态配置：在配置文件中、在命令行参数中、通过环境变量传递
  >
  > `动态配置`包含定义系统如何处理请求的所有配置内容，这些配置是可以改变的，而且是无缝热更新的，没有任何请求中断或连接损耗。
  >
  > ps: 具体可参考文章：[一文搞懂 Traefik2.1 的使用](https://www.qikqiak.com/post/traefik-2.1-101/)

+ 创建域名证书验证

  > 从域名服务商那里可以免费申请域名的tls证书，将其重名为``tls.crt``和``tls.key``
  >
  > 然后使用下面命令生成k8s的域名验证信息， ``domain-tls``为名称，后续会用到，可自行定义

  ```shell
  kubectl create secret tls domain-tls --cert=tls.crt --key=tls.key
  ```

+ 获取traefik安装文件

  ```shell
  git clone https://github.com/jageros/treafik-profile.git
  ```

+ 修改文件配置

  > 拉取到的仓库有四个yaml文件，其中crd.yaml和rbac.yaml两个文件是不需要修改的，其他两个需要做如下修改

  ```yaml
  # deployment.yaml
  #通过kubectl get nodes命令可以查看节点名称
  ...
  nodeSelector:
  
        kubernetes.io/hostname: master  # master必须为master节点的名称，根据自己的修改
  ...
  
  
  # dashboard.yaml
  ...
  routes:
     - match: Host(`www.domain.com`)  # 域名要改成自己的域名
   
  ...
  ...
  ...
   
    tls:
      secretName: domain-tls # 这个即上面生成域名验证信息的名称，需要改成相应的
  ```

+ 安装traefik

  ```shell
  # 通过shell脚本直接安装即可
  sh apply.sh
  ```

+ 检查traefik是否正常运行

  ```shell
  [root@master ~]# kubectl get pods -n kube-system
  NAME                             READY   STATUS    RESTARTS   AGE
  coredns-58cc8c89f4-4qq8h         1/1     Running   14         27d
  coredns-58cc8c89f4-vqj7f         1/1     Running   14         27d
  etcd-master                      1/1     Running   16         27d
  kube-apiserver-master            1/1     Running   16         27d
  kube-controller-manager-master   1/1     Running   18         27d
  kube-flannel-ds-26rkn            1/1     Running   18         27d
  kube-flannel-ds-68fv6            1/1     Running   16         27d
  kube-flannel-ds-n827d            1/1     Running   14         27d
  kube-flannel-ds-w4w2c            1/1     Running   16         27d
  kube-proxy-6rnfh                 1/1     Running   14         27d
  kube-proxy-m4ck5                 1/1     Running   14         27d
  kube-proxy-rzb4h                 1/1     Running   16         27d
  kube-proxy-sc5px                 1/1     Running   14         27d
  kube-scheduler-master            1/1     Running   18         27d
  traefik-78878774df-vzsc4         1/1     Running   4          11d
  ```

+ 检查是否可外网访问

  > 在域名解析中添加解析到master的IP，然后访问域名，看是否可以访问到dashboard UI（如图所示）

  ![dashborad](https://www.qikqiak.com/k8strain/assets/img/network/traefik-2.1.1-dashboard-demo.jpg)

## 三、部署应用

### 1、编译应用上传到docker仓库

+ 修改编译脚本配置信息

  > 编译脚本为项目中的build_all.sh，build_item.sh和Dockerfile文件
  >
  > 修改build_all.sh中的版本号和build_item.sh中的仓库地址
  >
  > 然后执行sh build_all.sh编译即可

+ 修改服务应用的yaml文件

  > 应用的yaml文件在项目目录下的k8s文件夹下面，需要把镜像的版本号修改成对应build_all.sh文件中的
  >
  > 域名和验证的tls名称也要改成对应的

### 2、部署应用

+ 先准备好Etcd、Redis、MongoDB和MySQL，并把配置写入到etcd中

+ 把应用的yaml文件打包上传到master机器上
+ 通过执行``apply.sh``脚本即可让应用跑起来

### 3、 测试服务是否正常

+ 通过dashborad ui查看路由是否正常
+ 通过``kubectl get pods -n yinyin``命令查看pod状态是否正常

### 4、更新应用

+ 版本号递增，然后修改yaml中的版本号
+ 应用：``kubectl apply -f xxx.yaml``

### 5、停止应用

```shell
sh delete.sh
```

## 参考

+ [kube-flannel.yml](https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml)
+ [traefik教程](https://www.qikqiak.com/post/ingress-traefik1/)