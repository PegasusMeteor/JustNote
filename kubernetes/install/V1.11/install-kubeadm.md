## 安装kubeadm

<img src="https://raw.githubusercontent.com/cncf/artwork/master/kubernetes/certified-kubernetes/versionless/color/certified-kubernetes-color.png" align="right" width="150px">本次将主要介绍如何安装 `kubeadm` 工具.如果对安装过程已经比较熟悉了的话，可以跳转到[使用kubeadm创建单个master的集群](create-cluster-kubeadm).

## 本章目录

- 开始之前
- 确认每个node的 MAC 地址 以及 product_uuid 是唯一的
- 检查网络
- 检查要求的端口
- 安装 docker
- 安装 kubeadm, kubelet and kubectl
- 配置在master节点上被kubelet 使用的cgroup-driver  
- 问题排查
- 下一步


## 开始之前

* 支持的运行环境:
  - Ubuntu 16.04+
  - Debian 9
  - CentOS 7
  - RHEL 7
  - Fedora 25/26 (best-effort)
  - HypriotOS v1.0.1+
  - Container Linux (tested with 1576.4.0)
* 每台机器 2G 以上的RAM (至少要留一点给其他应用程序使用)
* 2 CPUs  或更多
* 集群中所有的设备保持完全网络联通性 (public 或者 private 网络 都可以)
* 每个节点具有唯一的 hostname, MAC address, and product_uuid . 
* 确认需要的端口已经开启.
* 关闭Swap.  **必须** 禁用swap,否则 kubelet 可能运行不正常. 

## 确认每个node的 MAC 地址 以及 product_uuid 是唯一的

* 可以通过 `ip link` or `ifconfig -a` 命令获取 网卡的 MAC address .
* 通过 `sudo cat /sys/class/dmi/id/product_uuid` 命令获取product_uuid.

通常物理网卡会有唯一的MAC 地址，但是如果是复制的虚拟机却有可能会相同. Kubernetes 会使用 hostname, MAC address, and product_uuid(集群中必须唯一)这些值来唯一标识集群中的节点.如果这些值对每个节点来说，不是唯一的，那么安装过程可能会
[失败](https://github.com/kubernetes/kubeadm/issues/31).

## 检查网络适配器

如果网络环境中有多个网络适配器，并且Kubernetes组件不能访问到默认路由，建议在路由表中添加路由以便Kubernetes集群能够通过访问到合适的网络适配器.

## 检查要求的端口

### Master node(s)

| Protocol | Direction | Port Range | Purpose                 | Used By                   |
|----------|-----------|------------|-------------------------|---------------------------|
| TCP      | Inbound   | 6443*      | Kubernetes API server   | All                       |
| TCP      | Inbound   | 2379-2380  | etcd server client API  | kube-apiserver, etcd      |
| TCP      | Inbound   | 10250      | Kubelet API             | Self, Control plane       |
| TCP      | Inbound   | 10251      | kube-scheduler          | Self                      |
| TCP      | Inbound   | 10252      | kube-controller-manager | Self                      |

### Worker node(s)

| Protocol | Direction | Port Range  | Purpose               | Used By                 |
|----------|-----------|-------------|-----------------------|-------------------------|
| TCP      | Inbound   | 10250       | Kubelet API           | Self, Control plane     |
| TCP      | Inbound   | 30000-32767 | NodePort Services**   | All                     |

**  NodePort Services 的默认端口范围.

标记为*的端口号都是可覆盖的,因此需要确保我们自己配置的端口是打开的。

尽管ETCD的端口已经包含在了Master节点的端口之中，但是我们也可以自己指定端口来搭建ETCD集群.

我们使用的pod组件的网络插件(后面会有介绍)也需要端口是打开的. 因为每种pod network plugin 所需要的端口不同，所以需要查看不同插件的文档以确定需要开放哪些端口.

## 安装 Docker 

在集群的每个节点上安装 Docker.
要求版本为 17.03 以上,  1.11, 1.12 and 1.13 版本支持得最好.
 17.06+ 的版本 _可能会好用_ , 但是还没有被 Kubernetes node 团队测试和确认过.
最好在Kubernetes发布说明中跟踪最新的经过验证的Docker版本。

使用root来安装docker. 或者使用 `sudo -i` 命令来切换自己的操作权限.

如果已经安装了符合版本的docker可以直接跳转到到下一个环节.
如果没有可以参考下面的步骤:

如果操作系统是 **"Ubuntu, Debian or HypriotOS"** 类型的,可以从
Ubuntu的 repositories中安装docker:

```bash
apt-get update
apt-get install -y docker.io
```
或者从Docker为 Ubuntu 以及 Debian 准备的repositories 中直接安装Docker CE 17.03

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
```

如果操作系统是 **"CentOS, RHEL or Fedora"** 的话，可以实际使用系统本身的库来安装Docker:

```bash
yum install -y docker
systemctl enable docker && systemctl start docker
```

如果本身就是 **"Container Linux"** 的话，直接启用Docker就可以:

```bash
systemctl enable docker && systemctl start docker
```


可以参考 [Docker官方安装向导](https://docs.docker.com/engine/installation/)
获取更多信息.


#### 使用阿里云安装docker

&emsp;&emsp;  国内由于各种网络环境的原因，导致安装第三方组件并不是一件容易的事情。虽然像CentOS的EPEL源中已经包含了docker，但是版本却不一定能够满足我们的要求。所以我们建议直接安装Docker-CE版本。  
&emsp;&emsp;  阿里云开源镜像站中提供了docker-ce的下载包可以直接去下面的地址去下载。[https://mirrors.aliyun.com/docker-ce/linux/](https://mirrors.aliyun.com/docker-ce/linux/)  
&emsp;&emsp;  然后直接使用下面的命令安装和启动就可以了。 

```bash
yum install -y docker
systemctl enable docker && systemctl start docker
```

## 安装 kubeadm, kubelet and kubectl

我们需要在我们的设备上安装下面的几个包:

* `kubeadm`: 引导kubernetes集群.

* `kubelet`: 在集群中的所有机器上运行的组件,用来启动pod或者container.  

* `kubectl`: 与kubernetes交互的命令行工具.

kubeadm **不会** 安装或者管理 `kubelet` 与 `kubectl` 这两个工具，所以我们在安装kubeadm时，需要确保他与Kubernetes 的control plane的版本相匹配.如果不这样的话，可能会由于版本不兼容导致一些未知的bug. 尽管, 
kubelet 与 control plane 之间的小版本差异是支持的,但是 kubelet的版本将落后于API server的版本. 例如1.7.0 的kubelets 会与 1.8.0 的 API server 运行良好，但反之却不一定。

> 注意，系统升级时将不会升级有关kubernetes的包，因为 kubeadm and Kubernetes 需要
[专门的升级过程]().

如果操作系统是 **"Ubuntu, Debian or HypriotOS"** 类型的,按照下面的命令安装.

```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

如果操作系统是 **"CentOS, RHEL or Fedora"** 的话  

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet
```

#### 使用阿里云安装Kubernetes

&emsp;&emsp;  同样是由于国内网络环境的原因，安装过程可能会产生不少的问题因此，我们建议直接使用阿里云提供的镜像库来进行安装。  
&emsp;&emsp;  阿里云开源镜像站中提供了了yum以及apt两种下载方式，选择适合自己的一种就可以。[https://mirrors.aliyun.com/kubernetes/](https://mirrors.aliyun.com/kubernetes/)  
&emsp;&emsp;  然后直接使用下面的命令安装和启动就可以了。 

```bash 
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet

```

  **注意:**

  - 必须使用 `setenforce 0` 停用掉SELinux,也可以直接修改 `/etc/selinux/config` 文件来禁用SELinux.因为pod要求容器能够访问物理机的文件系统。  
  
  - 一些 RHEL/CentOS 7 用户报告说iptables 可能会错误地过滤请求. 要确保配置文件`sysctl`中
    `net.bridge.bridge-nf-call-iptables` 参数值已经设置为 1 , 例如.
  
    ```bash
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sysctl --system
    ```



如果本身就是 **"Container Linux"** 的话，需要
安装 CNI插件 (pod组件的网络都要用到):

```bash
CNI_VERSION="v0.6.0"
mkdir -p /opt/cni/bin
curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-amd64-${CNI_VERSION}.tgz" | tar -C /opt/cni/bin -xz
```

安装 `kubeadm`, `kubelet`, `kubectl` 并将`kubelet` 配置成系统服务:

```bash
RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"

mkdir -p /opt/bin
cd /opt/bin
curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/amd64/{kubeadm,kubelet,kubectl}
chmod +x {kubeadm,kubelet,kubectl}

curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/kubelet.service" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service
mkdir -p /etc/systemd/system/kubelet.service.d
curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/10-kubeadm.conf" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

启用 `kubelet`:

```bash
systemctl enable kubelet && systemctl start kubelet
```



现在，kubelet每隔几秒钟就会重新启动，因为它在一个死循环中等待kubeadm告诉它该做什么。

## 在master节点上配置 kubelet 使用的 cgroup-driver

如果使用了Dokcer的话， kubeadm 会为kubelet 自动检测 docker 的 cgroup driver 
并且在运行的时候把它配置在`/var/lib/kubelet/kubeadm-flags.env` 这个文件中. _这一点与旧版本有很大的不同，旧版本需要我们手动修改这个配置_ .

如果使用了不同的容器运行时接口(Container Runtime Interface, CRI), 就需要在
`/etc/default/kubelet` 文件中手动配置 `cgroup-driver` 的值, 例如:

```bash
KUBELET_KUBEADM_EXTRA_ARGS=--cgroup-driver=<value>
```
这个文件中存储了kubelet的一些用户自定义参数，并且会被`kubeadm init` and `kubeadm join`这两个命令来读取并配置到kubelet中.

记住一点，**只有** 当我们的容器运行时接口(Container Runtime Interface, CRI)不是`cgroupfs`的时候，我们才需要手动修改，因为kubelet默认就是`cgroupfs`.



最后需要重启kubelet:

```bash
systemctl daemon-reload
systemctl restart kubelet
```

## 问题排查

如果安装 kubeadm的过程中出现了什么问题,, 可以参考[kubeadm故障排除](troubleshooting-kubeadm).

## 下一步

* [使用kubeadm创建单个master的集群](create-cluster-kubeadm)





