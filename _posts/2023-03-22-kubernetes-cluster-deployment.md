---
layout: post
title: Kubernetes 集群部署
date: 2023-03-22 15:17 +0000
categories:
- 开发
- 服务
tags:
- containerd
- kubernetes
pin: true
---

Kubernetes 集群部署实操，需要有一定的 Linux 操作基础和错误排查能力，小白慎入。

## 环境准备

| 主机名                            | CPU/内存     | 操作系统         | 备注                                         |
|:----------------------------------|:-------------|:-----------------|:---------------------------------------------|
| kubernetes-control-plane-1.home   | 2C/8G        | Debian 12        | _**1** 到 **N**（1、3、5、...）个_           |
| kubernetes-worker-1.home          | _同上_       | Debian 12        | _**1** 到 **N**（1、2、3、...）个_           |
| kubernetes-worker-2.home          | _同上_       | Debian 12        | _同上_                                       |

更新系统，安装依赖：

```bash
apt-get update
apt-get upgrade -y
apt-get install curl gpg lsb-release software-properties-common -y
```

## 容器环境配置

需要安装最新 **containerd**，需要从 **docker** 仓库安装。

```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list
apt-get update
apt-get install containerd.io -y
```

打开 SystemdCgroup 驱动：
```bash
mv /etc/containerd/config.toml{,.default}
containerd config default | sed 's|^\(\s\+SystemdCgroup\s*=\s*\).*$|\1true|g' | tee /etc/containerd/config.toml
```

系统内核设置：
```bash
cat <<EOF | tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
modprobe overlay
modprobe br_netfilter
```

网络配置：
```bash
cat <<EOF | tee /etc/sysctl.d/99-kubernetes-k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl --system
systemctl restart containerd
systemctl enable containerd
```

## Kubernetes 安装

安装 Kubernetes 工具：
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmour -o /etc/apt/trusted.gpg.d/cgoogle.gpg
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
apt update
apt install kubelet kubeadm kubectl -y
apt-mark hold kubelet kubeadm kubectl
```

初始化控制面板节点：
```bash
kubeadm init --control-plane-endpoint "kubernetes-control-plane-1.home:6443" --upload-certs --pod-network-cidr=10.10.0.0/16
```
> 有多个控制面板节点时，***\--control-plane-endpoint*** 应该指定为控制面板的负载均衡服务，负载均衡服务可以通过 **HAProxy** 或同类软件实现。
{: .prompt-tip }

配置 kubectl 连接集群：
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

初始化工作节点：
```bash
kubeadm join kubernetes-control-plane-1.home:6443 --token anz5m5.uduchc1gc8d64fqa \
	--discovery-token-ca-cert-hash sha256:520baed040fc666718f0f37ac1596b9e92c96efbdea89972f0f4cd46eb0a0794
```

安装网络组件：
```bash
kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calico.yaml
```

## 其它

### 安装 Kubernetes Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

创建登录账号和 Token：
```bash
kubectl create serviceaccount dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
kubectl -n kube-system create token dashboard-admin
```
了解更多：<https://github.com/kubernetes/dashboard>

### 互通 Kubernetes 与本地网络
```bash
route add 10.10.0.0 mask 255.255.0.0 192.168.1.11
```
> - **10.10.0.0 mask 255.255.0.0** 是 Kubernetes 集群内部网段设置，对应：**\--pod-network-cidr=10.10.0.0/16**；
> - **192.168.1.11** 是 Kubernetes 控制面板或工作节点 IP 地址，任选一个即可；
> - route 可添加多个网段设置，可以本机设置，也可以在路由器设置，建议在路由器设置，这样局域网全局生效。
{: .prompt-info }
