---
title: kubernetes-kubeadm
date: 2018-06-26 09:44:28
tags:
---
<!-- ‎
    修改时间：2018‎年‎6‎月‎27‎日，‏‎14:12:15
    Author: liluyang
-->


# 使用kubeadm快速构建Kubernetes集群

本文将介绍如何在 Ubuntu server 16.04 版本上安装 kubeadm，并利用 kubeadm 快速的在 Ubuntu Server 16.04 上构建一个 Kubernetes 的基础集群，用来做学习和测试用途等，当前（2018-06-26）最新的版本是 1.10.4。参考文档包括 Kubernetes 官方网站的[ kubeadm 安装文档](https://kubernetes.io/docs/tasks/tools/install-kubeadm/) 以及 [利用kubeadm创建集群](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) 这两个文档。

生产用途的环境，需要考虑各个组件的高可用，建议参考 Kubernetes 的官方的相关的安装文档。

## 前置要求

1. 熟悉基本的 Linux 操作
2. 了解 Docker、容器技术等相关概念，有一定 Docker 命令使用经验
3. 了解基础的网络概念
4. 有一定的虚拟机使用经验，了解 vmware，xshell 等软件更好

## 准备工作

Kubernetes 安装建议至少 2 台服务器或者虚拟机，每台服务器 4G 内存，2 个 CPU 核心以上，基本架构为 1 台 master 节点，1 台 Slave 节点。本次的整个安装过程将 vmware 虚拟机下的两台 Ubuntu server 16.04 服务器上，包括 kubeadm、Kubernetes、canal 网络的安装。节点信息如下:

|角色|主机名|IP地址|
|---|---|---|
|Master|ubuntu|192.168.85.134|
|Slave|ubuntu-1|192.168.85.133|

1.	默认方式安装 Ubuntu Server 16.04
2.	每个节点配置主机名映射

::: warning 注意
在集群的部署过程中一定要先修改两台主机的 hostname，防止主机重名。因为我们只有两台主机，所以我们将 Slave 节点的 主机名称修改为 ubuntu-1 即可，Master 节点的名称为 ubuntu 不变。
:::

修改节点名称：

``` bash
# 在 slave 节点查看 hostname
$ hostname
ubuntu
$ vi hostname
# 编辑主机名称为 ubuntu-1，重启之后上校
# 重启后查询主机名称
$ hostname
ubuntu-1
```

## 安装 Docker

安装 Docker 可以使用系统源的的 docker.io 软件包，目前的版本 1.13.1，我的系统里是已经安装好最新的版本了。

目前（2018.06.26）Docker-CE 的最新版本已经更新到了 `18.03.1-ce` 以上，但是由于 Kubernetes 并没有对最新的 Docker 版本进行校验，所以可能会产生未知的问题。所以目前建议使用 apt-get 官方默认下载源的 docker.io 安装包。安装 1.13.1 的版本。

``` bash
$ apt-get install docker.io
Reading package lists... Done
Building dependency tree       
Reading state information... Done
docker.io is already the newest version (1.13.1-0ubuntu1~16.04.2).
0 upgraded, 0 newly installed, 0 to remove and 210 not upgraded.
```

Kubernetes 对 Docker 的版本支持列表如下所示：

|Kubernetes版本|Docker版本|
|---|---|
|Kubernetes 1.9|Docker 1.11.2 to 1.13.1 and 17.03.x|
|Kubernetes 1.8|Docker 1.11.2 to 1.13.1 and 17.03.x|
|Kubernetes 1.7|Docker 1.10.3,  1.11.2,  1.12.6|
|Kubernetes 1.6|Docker 1.10.3,  1.11.2,  1.12.6|
|Kubernetes 1.5|Docker 1.10.3,  1.11.2,  1.12.3|

官方对于 Docker 支持列表的文档参考链接：
- **Kubernetes 1.5**: [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.5.md#external-dependency-version-information](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.5.md#external-dependency-version-information)

- **Kubernetes 1.6**: [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.6.md#external-dependency-version-information](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.6.md#external-dependency-version-information)

- **Kubernetes 1.7**: [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.7.md#external-dependency-version-information](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.7.md#external-dependency-version-information)

- **Kubernetes 1.8**: [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.8.md#external-dependencies](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.8.md#external-dependencies)

- **Kubernetes 1.9**: [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.9.md#external-dependencies](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.9.md#external-dependencies)

## 在所有节点上安装 kubeadm

查看 apt 安装源，并且修改安装源配置文件，添加如下配置，使用阿里云的系统和 Kubernetes 的源。

``` bash
$ cat /etc/apt/sources.list
# 系统安装源
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
# kubeadm及kubernetes组件安装源
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main
```

修改完成后更新下载源，忽略不理会 gpg 的警告信息。

``` bash
# apt-get update
Hit:1 http://mirrors.aliyun.com/ubuntu xenial InRelease
Hit:2 http://mirrors.aliyun.com/ubuntu xenial-updates InRelease
Hit:3 http://mirrors.aliyun.com/ubuntu xenial-backports InRelease
Get:4 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease [8,993 B]
Ign:4 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease
Fetched 8,993 B in 0s (20.7 kB/s)
Reading package lists... Done
W: GPG error: https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 6A030B21BA07F4FB
W: The repository 'https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease' is not signed.
N: Data from such a repository can't be authenticated and is therefore potentially dangerous to use.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```

::: warning 注意
建议使用 Ubuntu 16.04 版本的系统。Ubuntu 18.04 的系统在进行这一步的更新下载源会失败。更新源的警告 Warning 会变成 Error 导致更新源不成功,无法使用 apt-get 包管理工具进行后续的 kubeadm 等软件的下载。
:::

在 Ubuntu 18.04 系统中替代方法是可以使用 snap 包管理下载 kudeadm 等软件，然后修改环境变量 `PATH`,将 kubeadm 等软件的执行路径添加到其中即可，具体用法可以自行搜索。不过不依然建议使用这种方法。


## 使用 apt 强制安装 kubeadm，kubectl，kubelet 软件包。

``` bash
# apt-get install -y kubelet kubeadm kubectl --allow-unauthenticated
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  kubernetes-cni socat
The following NEW packages will be installed:
  kubeadm kubectl kubelet kubernetes-cni socat
0 upgraded, 5 newly installed, 0 to remove and 4 not upgraded.
Need to get 56.9 MB of archives.
After this operation, 410 MB of additional disk space will be used.
WARNING: The following packages cannot be authenticated!
  kubernetes-cni kubelet kubectl kubeadm
Authentication warning overridden.
Get:1 http://mirrors.aliyun.com/ubuntu xenial/universe amd64 socat amd64 1.7.3.1-1 [321 kB]
Get:2 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 kubernetes-cni amd64 0.6.0-00 [5,910 kB]
Get:3 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 kubelet amd64 1.10.1-00 [21.1 MB]
Get:4 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 kubectl amd64 1.10.1-00 [8,906 kB]
Get:5 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 kubeadm amd64 1.10.1-00 [20.7 MB]
Fetched 56.9 MB in 5s (11.0 MB/s)
Use of uninitialized value $_ in lc at /usr/share/perl5/Debconf/Template.pm line 287.
Selecting previously unselected package kubernetes-cni.
(Reading database ... 191799 files and directories currently installed.)
Preparing to unpack .../kubernetes-cni_0.6.0-00_amd64.deb ...
Unpacking kubernetes-cni (0.6.0-00) ...
Selecting previously unselected package socat.
Preparing to unpack .../socat_1.7.3.1-1_amd64.deb ...
Unpacking ....
....
```

由于演示代码安装的 kubernetes 的版本是 1.9.1，这对于最新版本的 kubeadm 来说是不兼容的，因此我们可以在下载 kubeadm 的过程中指定版本安装，下面的示例中我们安装的版本是 1.9.0。使用这个版本我们可以正常的安装 kubernetes 1.9.1。

``` bash
# apt-cache madison kubeadm
   kubeadm |  1.10.5-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.4-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages

   ······

   kubeadm |   1.9.1-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.0-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |  1.8.14-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   
   ······

   kubeadm |   1.6.2-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.1-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |   1.5.7-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
# apt-get install kubeadm=1.9.0-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
kubeadm is already the newest version (1.9.0-00).
0 upgraded, 0 newly installed, 0 to remove and 209 not upgraded.
# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.0", GitCommit:"925c127ec6b946659ad0fd596fa959be43f0cc05", GitTreeState:"clean", BuildDate:"2017-12-15T20:55:30Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
```

到此为止，kubeadm 已经安装完成，我们可以使用它来快速安装部署 Kubernetes 集群了。


## 使用 kubeadm 安装 Kubernetes 集群

### 下载安装 Docker 所需的镜像列表

如果你的网络已经翻墙，那么跳过本节，直接进入下一节执行 `kubeadm init` 初始化 master 节点

在执行 kubeadm 命令时系统会自动访问 Google Cloud Platform 去下载安装 kubernetes 集群的核心镜像。但是由于 `GFW` 的原因，导致我们无法正常的访问 Google Cloud Platform，所以就需要换一种方法去下载镜像。这里叫大家一种下载镜像的方式


``` bash
echo 下载镜像
docker pull xiaoyaolangzi/etcd-amd64:3.1.10
docker pull xiaoyaolangzi/kube-apiserver-amd64:v1.9.1
docker pull xiaoyaolangzi/kube-controller-manager-amd64:v1.9.1
docker pull xiaoyaolangzi/kube-scheduler-amd64:v1.9.1

docker pull xiaoyaolangzi/k8s-dns-dnsmasq-nanny-amd64:1.14.7
docker pull xiaoyaolangzi/k8s-dns-kube-dns-amd64:1.14.7
docker pull xiaoyaolangzi/k8s-dns-sidecar-amd64:1.14.7
docker pull xiaoyaolangzi/kube-proxy-amd64:v1.9.1
docker pull xiaoyaolangzi/pause-amd64:3.0

docker pull xiaoyaolangzi/metrics-server-amd64:v0.2.0


echo 修改标签
docker tag xiaoyaolangzi/etcd-amd64:3.1.10 gcr.io/google_containers/etcd-amd64:3.1.10
docker tag xiaoyaolangzi/kube-apiserver-amd64:v1.9.1 gcr.io/google_containers/kube-apiserver-amd64:v1.9.1
docker tag xiaoyaolangzi/kube-controller-manager-amd64:v1.9.1 gcr.io/google_containers/kube-controller-manager-amd64:v1.9.1
docker tag xiaoyaolangzi/kube-scheduler-amd64:v1.9.1 gcr.io/google_containers/kube-scheduler-amd64:v1.9.1

docker tag xiaoyaolangzi/k8s-dns-dnsmasq-nanny-amd64:1.14.7 gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.7
docker tag xiaoyaolangzi/k8s-dns-kube-dns-amd64:1.14.7 gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.7
docker tag xiaoyaolangzi/k8s-dns-sidecar-amd64:1.14.7 gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.7
docker tag xiaoyaolangzi/kube-proxy-amd64:v1.9.1 gcr.io/google_containers/kube-proxy-amd64:v1.9.1
docker tag xiaoyaolangzi/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0

docker tag xiaoyaolangzi/metrics-server-amd64:v0.2.0 gcr.io/google_containers/metrics-server-amd64:v0.2.0


echo 删除镜像
docker rmi xiaoyaolangzi/etcd-amd64:3.1.10
docker rmi xiaoyaolangzi/kube-apiserver-amd64:v1.9.1
docker rmi xiaoyaolangzi/kube-controller-manager-amd64:v1.9.1
docker rmi xiaoyaolangzi/kube-scheduler-amd64:v1.9.1

docker rmi xiaoyaolangzi/k8s-dns-dnsmasq-nanny-amd64:1.14.7
docker rmi xiaoyaolangzi/k8s-dns-kube-dns-amd64:1.14.7
docker rmi xiaoyaolangzi/k8s-dns-sidecar-amd64:1.14.7
docker rmi xiaoyaolangzi/kube-proxy-amd64:v1.9.1
docker rmi xiaoyaolangzi/pause-amd64:3.0

docker rmi xiaoyaolangzi/metrics-server-amd64:v0.2.0
```

复制上面的脚本，新建一个 1.9.1.sh 的脚本，将内容复制进去，执行该脚本下载镜像

``` bash
# 修改执行权限
$ chmod 755 ./1.9.1.sh
# 执行脚本
$ ./1.9.1.sh
```

下载完成后使用查看下载完成的镜像列表，如果最后下载完成的镜像列表如下图所示，那么证明镜像列表已经下载成功了

``` bash
# docker images
REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE
gcr.io/google_containers/metrics-server-amd64            v0.2.0              05ddb332f1ff        5 months ago        96.5 MB
gcr.io/google_containers/pause-amd64                     3.0                 3773f3dd7d9b        5 months ago        747 kB
gcr.io/google_containers/kube-scheduler-amd64            v1.9.1              7f4072f8640c        5 months ago        62.7 MB
gcr.io/google_containers/kube-proxy-amd64                v1.9.1              d6b97770e509        5 months ago        109 MB
gcr.io/google_containers/kube-controller-manager-amd64   v1.9.1              94e3348da21f        5 months ago        138 MB
gcr.io/google_containers/kube-apiserver-amd64            v1.9.1              36589a515579        5 months ago        210 MB
gcr.io/google_containers/k8s-dns-sidecar-amd64           1.14.7              da641877cb23        5 months ago        42 MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64          1.14.7              80db1c7f1052        5 months ago        50.3 MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64     1.14.7              cdb0a2151300        5 months ago        41 MB
gcr.io/google_containers/etcd-amd64                      3.1.10              83e68e9d1d4a        5 months ago        193 MB
```

### 使用kubeadm初始化master节点

因为使用要使用 canal，因此需要在初始化时加上网络配置参数,设置 kubernetes 的子网为 10.244.0.0/16，注意此处不要修改为其他地址，因为这个值与后续的 canal 的 yaml 值要一致，如果修改，请一并修改。

这个下载镜像的过程涉及翻墙，因为会从 [gcr.io](//gcr.io) 的站点下载容器镜像。。。（如果大家翻墙不方便的话，请回头看上一个章节）。

如果有能够连接 [gcr.io](//gcr.io) 站点的网络，那么整个安装过程非常简单，直接运行以下命令即可。

``` bash
# kubeadm init --kubernetes-version=1.9.1 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.9.1
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
	[WARNING FileExisting-crictl]: crictl not found in system path
[preflight] Starting the kubelet service
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [ubuntu kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.244.0.1 192.168.85.134]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 29.504162 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node ubuntu as master by adding a label and a taint
[markmaster] Master ubuntu tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 7e9211.94f5426a55b22eb3
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 7e9211.94f5426a55b22eb3 192.168.85.134:6443 --discovery-token-ca-cert-hash sha256:cb009a5dde6b127d4844cffe04163a0ce07f10557cd0c2ccfa4c51eb5536ab02

```

执行如下命令来配置 kubectl。

``` bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


这样 Master 的节点就配置好了，并且可以使用 kubectl 来进行各种操作了。

::: warning 注意
如果在部署过程中依然出现了卡在这一步的情况，那么可能是镜像包的名称没有修改正确，导致 kubeadm 无法找到安装 kubernetes 所需要的镜像，依然去 gcr.io 下载镜像，而我们的网络又没有翻墙，导致卡在这一步无法正常安装成功。
:::

解决办法如下：在执行完成 kubeadm init 命令后，本地的文件系统中就生成了以下几个核心组件的配置文件，这几个配置文件中描述了需要使用的镜像的名称和版本。使用 cat 分别输出这四个文件的内容：

``` bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
cat /etc/kubernetes/manifests/kube-scheduler.yaml
cat /etc/kubernetes/manifests/etcd.yaml
```
正常情况配置文件中的 yaml 文件中的 image 字段应该是 `image: gcr.io/google_containers/kube-controller-manager-amd64:v1.9.1` 等等这样的结构，但是之前我自己在部署的过程中遇到过一次需要下载的镜像名称不是这种格式而是 `image: k8s.gcr.io/kube-controller-manager-amd64:v1.9.1`，这个时候我们需要接着执行以下命令，修改镜像的名称

``` bash
# docker tag gcr.io/google_containers/kube-apiserver-amd64:v1.9.1 k8s.gcr.io/kube-apiserver-amd64:v1.9.1
# docker tag gcr.io/google_containers/kube-controller-manager-amd64:v1.9.1 k8s.gcr.io/kube-controller-manager-amd64:v1.9.1
# docker tag gcr.io/google_containers/kube-scheduler-amd64:v1.9.1 k8s.gcr.io/kube-scheduler-amd64:v1.9.1
# docker tag gcr.io/google_containers/etcd-amd64:3.1.12 k8s.gcr.io/etcd-amd64:3.1.12
```

修改完成之后，再次执行 `kubeadm init` 命令，稍等片刻 kubernetes 即可安装成功

### 安装 canal 网络插件

从 canal 官方文档参考，如下网址下载 2 个文件并且安装，其中一个是配置 canal 的 RBAC 权限，一个是部署 canal 的 DaemonSet。

``` bash
# kubectl apply -f  https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
clusterrole.rbac.authorization.k8s.io "calico" created
clusterrole.rbac.authorization.k8s.io "flannel" created
clusterrolebinding.rbac.authorization.k8s.io "canal-flannel" created
clusterrolebinding.rbac.authorization.k8s.io "canal-calico" created
```

``` bash
# kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/canal/canal.yaml
configmap "canal-config" created
daemonset.extensions "canal" created
customresourcedefinition.apiextensions.k8s.io "felixconfigurations.crd.projectcalico.org" created
customresourcedefinition.apiextensions.k8s.io "bgpconfigurations.crd.projectcalico.org" created
customresourcedefinition.apiextensions.k8s.io "ippools.crd.projectcalico.org" created
customresourcedefinition.apiextensions.k8s.io "clusterinformations.crd.projectcalico.org" created
customresourcedefinition.apiextensions.k8s.io "globalnetworkpolicies.crd.projectcalico.org" created
customresourcedefinition.apiextensions.k8s.io "networkpolicies.crd.projectcalico.org" created
serviceaccount "canal" created
```


等待所有节点运行成功之后查看 canal 的安装状态，可以看到 canal 和 kube-dns 都已经运行正常，一个功能正常的 matser 节点和必须的网络插件就部署完毕了。

``` bash
# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
ubuntu    Ready     master    5m        v1.9.0
# kubectl get pod -n kube-system -o wide
NAME                             READY     STATUS    RESTARTS   AGE       IP               NODE
canal-4n2tw                      3/3       Running   0          1m        192.168.85.134   ubuntu
etcd-ubuntu                      1/1       Running   0          3m        192.168.85.134   ubuntu
kube-apiserver-ubuntu            1/1       Running   0          3m        192.168.85.134   ubuntu
kube-controller-manager-ubuntu   1/1       Running   0          3m        192.168.85.134   ubuntu
kube-dns-6f4fd4bdf-gpvh4         3/3       Running   0          3m        10.244.0.5       ubuntu
kube-proxy-kbdvk                 1/1       Running   0          3m        192.168.85.134   ubuntu
kube-scheduler-ubuntu            1/1       Running   0          3m        192.168.85.134   ubuntu   ubuntu-master
```

## 将Slave节点加入集群

不同于 Master 节点， Slave 节点所需的镜像较少，在 Slave 节点执行以下命令下载镜像

``` bash
# download
docker pull xiaoyaolangzi/kube-proxy-amd64:v1.9.1
docker pull xiaoyaolangzi/pause-amd64:3.0

# rename tag
docker tag xiaoyaolangzi/kube-proxy-amd64:v1.9.1 gcr.io/google_containers/kube-proxy-amd64:v1.9.1
docker tag xiaoyaolangzi/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0

# delete
docker rmi xiaoyaolangzi/kube-proxy-amd64:v1.9.1
docker rmi xiaoyaolangzi/pause-amd64:3.0
```

在 Slave 节点执行如下的命令,将 Slave 节点加入集群。正常的返回信息如下：

``` bash
#kubeadm join --token 7e9211.94f5426a55b22eb3 192.168.85.134:6443 --discovery-token-ca-cert-hash sha256:cb009a5dde6b127d4844cffe04163a0ce07f10557cd0c2ccfa4c51eb5536ab02
[preflight] Running pre-flight checks.
    [WARNING FileExisting-crictl]: crictl not found in system path
Suggestion: go get github.com/kubernetes-incubator/cri-tools/cmd/crictl
[discovery] Trying to connect to API Server "192.168.0.200:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://192.168.0.200:6443"
[discovery] Requesting info from "https://192.168.0.200:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "192.168.0.200:6443"
[discovery] Successfully established connection with API Server "192.168.0.200:6443"

This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```

::: warning 注意
此命令是你在 Master 节点执行 kubeadm init 时返回的命令，此命令只是一个示例，直接复制上面的命令 ip, token 等信息都是不匹配的。
:::

返回 Master 节点，等待节点加入完毕后查看节点状态

``` bash
# kubectl get nodes
NAME            STATUS     ROLES     AGE       VERSION
ubuntu-master   NotReady   master    10m       v1.9.0
ubuntu-1        NotReady   <none>    6m        v1.9.0
# kubectl get pod -n kube-system -o wide
NAME                                    READY     STATUS    RESTARTS   AGE       IP               NODE
canal-s8hks                             3/3       Running   0          21m       192.168.85.133   ubuntu-1
canal-wc4fh                             3/3       Running   0          28m       192.168.85.134   ubuntu
etcd-ubuntu                             1/1       Running   0          23m       192.168.85.134   ubuntu
kube-apiserver-ubuntu                   1/1       Running   0          23m       192.168.85.134   ubuntu
kube-controller-manager-ubuntu          1/1       Running   0          23m       192.168.85.134   ubuntu
kube-dns-6f4fd4bdf-pcdls                3/3       Running   0          29m       10.244.0.10      ubuntu
kube-proxy-fn69j                        1/1       Running   0          21m       192.168.85.133   ubuntu-1
kube-proxy-jjpz6                        1/1       Running   0          29m       192.168.85.134   ubuntu
kube-scheduler-ubuntu                   1/1       Running   0          23m       192.168.85.134   ubuntu
kubernetes-dashboard-5bd6f767c7-9msq6   1/1       Running   0          28m       10.244.0.9       ubuntu
```

返回 Slave 节点，查看下载的镜像列表

``` bash
# docker images
REPOSITORY                                  TAG                 IMAGE ID            CREATED             SIZE
quay.io/calico/node                         v3.0.8              6e991381712e        3 weeks ago         248 MB
quay.io/calico/cni                          v2.0.6              dbeb77ece97f        3 weeks ago         69.1 MB
gcr.io/google_containers/pause-amd64        3.0                 3773f3dd7d9b        5 months ago        747 kB
gcr.io/google_containers/kube-proxy-amd64   v1.9.1              d6b97770e509        5 months ago        109 MB
quay.io/coreos/flannel                      v0.9.1              2b736d06ca4c        7 months ago        51.3 MB
```

可以发现，Slave 节点也下载了我们必须使用的镜像。到此为止，我们已经成功的在本地虚拟机上搭建起来了一个 Kubernetes 测试集群。这个测试集群拥有完整的功能，已经可以供我们正常使用 Kubernetes 了。

## 可能需要使用到的命令

1. 让 Master 也运行 pod（默认 master 不运行 pod ）,这样在测试环境做是可以的，不建议在生产环境如此操作。

``` bash
# kubectl taint nodes --all node-role.kubernetes.io/master-
node "ubuntu-master" untainted
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```

2. 重置集群状态 `kubeadm reset`，不是每次执行 `kubeadm init` 或者 `kubeadm join` 命令都能保证执行成功的。所以在这些命令执行失败之后我们需要 `kubeadm reset` 来重置一下集群的状态。

``` bash
# kubeadm reset
[preflight] Running pre-flight checks.
[reset] Stopping the kubelet service.
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers.
[reset] No etcd manifest found in "/etc/kubernetes/manifests/etcd.yaml". Assuming external etcd.
[reset] Deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes]
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
```
3. 使用 `kubectl describe --namespace=kube-system pod <pod_name>` 查找错误原因，如果在部署过程中发现有状态错误的 Pod，则可以执行该命令来查看错误原因，比较常见的问题可能是镜像没有下载。

4. 使用 `kebeadm help` 和 `kubectl help` 来查看对应的命令行工具的使用文档

``` bash
# kubeadm help
kubeadm: easily bootstrap a secure Kubernetes cluster.

    ┌──────────────────────────────────────────────────────────┐
    │ KUBEADM IS CURRENTLY IN BETA                             │
    │                                                          │
    │ But please, try it out and give us feedback at:          │
    │ https://github.com/kubernetes/kubeadm/issues             │
    │ and at-mention @kubernetes/sig-cluster-lifecycle-bugs    │
    │ or @kubernetes/sig-cluster-lifecycle-feature-requests    │
    └──────────────────────────────────────────────────────────┘

Example usage:

    Create a two-machine cluster with one master (which controls the cluster),
    and one node (where your workloads, like Pods and Deployments run).

    ┌──────────────────────────────────────────────────────────┐
    │ On the first machine:                                    │
    ├──────────────────────────────────────────────────────────┤
    │ master# kubeadm init                                     │
    └──────────────────────────────────────────────────────────┘

    ┌──────────────────────────────────────────────────────────┐
    │ On the second machine:                                   │
    ├──────────────────────────────────────────────────────────┤
    │ node# kubeadm join <arguments-returned-from-init>        │
    └──────────────────────────────────────────────────────────┘

    You can then repeat the second step on as many other machines as you like.

Usage:
  kubeadm [command]

Available Commands:
  alpha       Experimental sub-commands not yet fully functional.
  completion  Output shell completion code for the specified shell (bash or zsh).
  config      Manage configuration for a kubeadm cluster persisted in a ConfigMap in the cluster.
  init        Run this command in order to set up the Kubernetes master.
  join        Run this on any machine you wish to join an existing cluster
  reset       Run this to revert any changes made to this host by 'kubeadm init' or 'kubeadm join'.
  token       Manage bootstrap tokens.
  upgrade     Upgrade your cluster smoothly to a newer version with this command.
  version     Print the version of kubeadm

Use "kubeadm [command] --help" for more information about a command.
```

## 参考文章连接

1. **官方**：[kubeadm 安装文档](https://kubernetes.io/docs/tasks/tools/install-kubeadm/)
2. **官方**：[利用 kubeadm 创建集群](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
3. **官方**：[kubeadm 使用教程](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/)
3. **官方**：[kubernetes 更新日志](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md)
4. **官方**：[在 Ubuntu 系统中下载 Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
5. **第三方**：[Kubernetes Handbook](https://jimmysong.io/kubernetes-handbook/)
6. **第三方**：[Kubernetes 对 Docker 的版本支持列表](https://blog.csdn.net/CSDN_duomaomao/article/details/79171027)
6. **第三方**：[如何确定 kubernetes 依赖的各个组件版本？](http://blog.51cto.com/dangzhiqiang/2107424?utm_source=oschina-app)

