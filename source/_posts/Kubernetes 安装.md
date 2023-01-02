---
title: Kubernetes 安装
urlname: swcolmpr9f0ttwqd
date: '2022-12-31 14:01:53 +0800'
tags: []
categories: []
---

tags: []
categories: [Kubernetes]
cover: ''

---

## 安装工具

### Kubectl

> 让你可以对 Kubernetes 集群运行命令。
> 你可以使用 kubectl 来部署应用、监测和管理集群资源以及查看日志。

#### Linux 安装设置

##### curl

```bash
#下载最新版
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

#安装
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#测试
kubectl version --client

#详细信息
kubectl version --client --output=yaml
```

##### 包管理工具

```bash
#更新包
sudo apt-get update
sudo apt-get install -y ca-certificates curl

#如果你使用 Debian 9（stretch）或更早版本，则你还需要安装 apt-transport-https
sudo apt-get install -y apt-transport-https

#下载 Google Cloud 公开签名秘钥
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

#添加 Kubernetes apt 仓库
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

#更新 apt 包索引，使之包含新的仓库并安装 kubectl
sudo apt-get update
sudo apt-get install -y kubectl
```

#### mac 安装

```bash
brew install kubectl
#或者
brew install kubernetes-cli

#测试
kubectl version --client
```

### kind

> [kind](https://kind.sigs.k8s.io/docs/) 让你能够在本地计算机上运行 Kubernetes。
> kind [快速入门](https://kind.sigs.k8s.io/docs/user/quick-start/)页面展示了开始使用 kind 所需要完成的操作。

### minikube

> 与 kind 类似，[minikube](https://minikube.sigs.k8s.io/) 是一个工具， 能让你在本地运行 Kubernetes。
> minikube 在你的个人计算机（包括 Windows、macOS 和 Linux PC）上运行一个一体化（all-in-one）或多节点的本地 Kubernetes 集群，以便你来尝试 Kubernetes 或者开展每天的开发工作。
> 只是一个 K8S 集群模拟器，只有一个节点的集群，只为测试用，master 和 worker 都在一起。
> [安装方法](https://minikube.sigs.k8s.io/docs/start/)

#### 二进制安装

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

#### Homebrew

```bash
brew install minikube
#如果安装失败，删除重新安装
brew unlink minikube
brew link minikube

```

#### 命令

```bash
# 启动集群
minikube start
# 查看节点。kubectl 是一个用来跟 K8S 集群进行交互的命令行工具
kubectl get node
# 停止集群
minikube stop
# 清空集群
minikube delete --all
# 安装集群可视化 Web UI 控制台
minikube dashboard
```

### kubeadm

> 你可以使用 [kubeadm](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/) 工具来创建和管理 Kubernetes 集群。 该工具能够执行必要的动作并用一种用户友好的方式启动一个可用的、安全的集群。

### 关于 K8s mac 电脑一直处于 starting 状态解决

> 由于国内访问 Docker Hub 网速太慢，镜像无法成功拉取，导致 Kubernetes 一直处于 starting 状态。

```bash
git clone git@github.com:hummerstudio/k8s-docker-desktop-for-mac.git

步骤2：执行根目录下load_images.sh脚本即可正常下载镜像：

sh load_images.sh

```
