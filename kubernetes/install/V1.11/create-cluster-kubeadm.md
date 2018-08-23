## 使用kubeadm创建单个master的集群

<img src="https://raw.githubusercontent.com/cncf/artwork/master/kubernetes/certified-kubernetes/versionless/color/certified-kubernetes-color.png" align="right" width="150px">**kubeadm** 是帮助我们创建最小可用kuernetes集群的最佳实践.  使用 kubeadm 创建的集群应该通过 [Kubernetes 一致性测试](https://kubernetes.io/blog/2017/10/software-conformance-certification). Kubeadm 还支持其他的控制集群的功能，例如升级，降级，管理[bootstrap tokens]等. 

因为 kubeadm 可以安装在各种类型的设备上 (例如 笔记本, 服务器, 
树莓派等等), 所以它也很适用于一些自动化工具，例如Terraform 或者 Ansible.

kubeadm 的简单性意味着他可以用于非常广泛的场景:

- 新用户可以从kubeadm开始，第一次尝试使用Kubernetes.
- 熟悉Kubernetes的用户可以使用kubeadm来配置集群并测试他们的应用程序.
- 还可以将 kubeadm 作为复杂系统的一个子模块,与其他需要安装的工具一起组成一个更大的工程中.

kubeadm 可以让新用户很方便的尝试kubernetes, 对于用户来说，可以很方便的测试他们的应用并且与集群无缝衔接,可能也是首次。同时kubeadm 还可以作为集成或者安装工具包的一个模块被打包到更大的工程中。


在支持 deb 或者 rpm 包的操作系统上，我们可以很容易的安装  _kubeadm_  .


### kubeadm 进展(是否成熟可用)

| Area                      | Maturity Level |
|---------------------------|--------------- |
| Command line UX           | beta           |
| Implementation            | beta           |
| Config file API           | alpha          |
| Self-hosting              | alpha          |
| kubeadm alpha subcommands | alpha          |
| CoreDNS                   | GA             |
| DynamicKubeletConfig      | alpha          |


kubeadm 里所有 **Beta** 阶段的特性在2018年就会发展到
**一般可用 (GA)** . 一些子特性, 例如 self-hosting
和  configuration file API 还在积极开发过程中. 创建集群的过程可能会随着工具的开发发生相应的变化,
但是总体上会趋于稳定. 
`kubeadm alpha` 状态的定义, 只在alpha 阶段进行支持.


### 支持时间表

kubernetes 通常每9个月发布一个版本，但是在这期间，如果发现了bug或者安全隐患的话就会发布一个新的分支版本。下面是最新的Kubernetes版本以及服务支持截止时间。这个时间表同样适用于 `kubeadm`.

| Kubernetes version | Release month  | End-of-life-month |
|--------------------|----------------|-------------------|
| v1.6.x             | March 2017     | December 2017     |
| v1.7.x             | June 2017      | March 2018        |
| v1.8.x             | September 2017 | June 2018         |
| v1.9.x             | December 2017  | September 2018    |
| v1.10.x            | March 2018     | December 2018     |
| v1.11.x            | June 2018      | March 2019        |


## 本章目录 


- 开始之前
- 目标
- 安装过程
- 移除
- 维护一个集群
- 浏览其他附加组件
- 问题排查


## 开始之前

- 一个或者多个能够安装deb包或者rpm包的设备, 例如Ubuntu 或者 CentOS 主机
- 至少 2 GB 的内存.
- master 节点上至少 双核 CPU 。
- 集群中所有节点网络互通，公网或者内网均可.
 

## 目标

* 安装单个master节点的集群或者 [高可用集群](https://kubernetes.io/docs/setup/independent/high-availability/)
* 在集群中安装 pod 网络，以便所有的pod节点能够互相通信。

## 安装过程

### 安装kubeadm

请查看 ["安装kubeadm"](install-kubeadm.html).

> **注意:** 如果已经安装了 kubeadm, 运行 `apt-get update &&
apt-get upgrade` 或者 `yum update` 去获取最新版本的 kubeadm.

在升级的过程中，kubelet会陷入死循环，每隔几秒钟重启一下，直到kubeadm通知它要做什么。死循环是正常并且有必要的。
初始化了 master节点之后, kubelet 会恢复正常.  


### 初始化 master 节点

Master节点是  control plane 组件运行的地方 , 包括了
etcd (集群数据库，自动注册与发现) 和 API server (与kubectl 命令行交互的).

1. 选择一个pod网络附加组件，并确认它是否需要在kubeadm初始化时传入相应的参数. 根据选择的第三方提供商的不同，有可能需要将 `--pod-network-cidr` 设置成第三方指定的值. 查看 [安装 pod 网络附件](#pod-network).
1. (可选) 除非另有说明, 否则 kubeadm 使用具有默认网关的网络接口来广播maste节点的IP. 如果要使用不同的网络接口,需要在执行 `kubeadm init` 命令时 指定 `--apiserver-advertise-address=<ip-address>` 参数. 如果要部署基于IPv6地址的 IPv6 Kubernetes 集群,就必须要指定IPv6的地址，例如 `--apiserver-advertise-address=fd00::101`
1. (可选) 运行`kubeadm init` 之前先运行 `kubeadm config images pull` 来确定 gcr.io registries 的联通性.   

接下来运行:

```bash
kubeadm init <args> 
```

### 更多信息


想要再次运行 `kubeadm init` , 首先要 [停止掉集群](#tear-down).

如果要将一个不同体系结构(architecture)的node假如到集群中，在这个节点上需要单独创建`kube-proxy` 和 `kube-dns`。因为这些组件的Docker 镜像不支持 多体系结构(multi-architecture)。


`kubeadm init` 命令首先会进行一系列的检测以确保这台机器适合运行kubernetes. 这些检测或输出warnings，并在出现errors的时候停止掉. 然后 `kubeadm init` 命令会下载集群控制台(cluster control plane)需要的各种组件. 需要几分钟的时间. 输出应该会像下面这个样子:

```none
[init] Using Kubernetes version: vX.Y.Z
[preflight] Running pre-flight checks
[kubeadm] WARNING: starting in 1.8, tokens expire after 24 hours by default (if you require a non-expiring token use --token-ttl 0)
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [kubeadm-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.138.0.4]
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
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests"
[init] This often takes around a minute; or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 39.511972 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node master as master by adding a label and a taint
[markmaster] Master master tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: <token>
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the addon options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```
为了非root用户能够更好的使用kubectl，可以执行下面的命令，这些内容 也是`kubeadm init` 执行过程中会提示的内容:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

另外, 如果已经是 `root` 用户了,可以执行下面的命令:

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

运行 `kubeadm join`命令将节点加入到集群中去 .

token 用于在master节点和新加入的node节点之间互相进行身份认证。这里token要安全保存，因为任何知道token的用户都能通过它认证并加入到集群中去。使用 `kubeadm token` 命令可以列举，创建和删除 token。

### 安装 POD 网络组件   


> **注意:** 本节包含关于安装和部署顺序的重要信息。在继续之前请仔细阅读。


因为pod要求互相之间能够正常通信，所以必须安装网络组件。  


**在发布应用程序之前，必须先将网络调通. 并且,  在网络没有安装好之前也不能启动CoreDNS.
kubeadm 只支持基于网络的容器网络接口(Container Network Interface (CNI)) (不支持 kubenet).**

有些项目使用CNI为Kubernetes pod提供网络，其中一些还支持网络策略。
- [CNI v0.6.0](https://github.com/containernetworking/cni/releases/tag/v0.6.0) 开始支持IPv6.
- [CNI bridge](https://github.com/containernetworking/plugins/blob/master/plugins/main/bridge/README.md) 和 [local-ipam](https://github.com/containernetworking/plugins/blob/master/plugins/ipam/host-local/README.md) 是 Kubernetes  1.9 版本中唯一支持IPv6的组件.

请注意，kubeadm默认设置了一个更安全的集群，并强制使用[RBAC](/docs/reference/access-authn-authz/rbac/).
确保你的网络清单支持RBAC.

可以通过下面的命令来安装pod network组件。:

```bash
kubectl apply -f <add-on.yaml>
```

每个集群中可以只安装一个 pod network.



根据下面不同的网络提供商选择不同的安装方式。


#### Calico  

可以点击 [Quickstart for Calico on Kubernetes](https://docs.projectcalico.org/latest/getting-started/kubernetes/), [Installing Calico for policy and networking](https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/calico),获取更多信息.

为了使网络策略正确工作, 在运行`kubeadm init`命令时需要指定 `--pod-network-cidr=192.168.0.0/16` . 注意 Calico 只支持 `amd64` 平台.

```shell
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

#### Canal    

Canal 使用Calico做网络策略，Flannel来进行网络转发。点击后面链接查看 Calico 文档 [official getting started guide](https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/flannel).

要使Canal更好地运行,在运行 `kubeadm init` 命令时需要指定 `--pod-network-cidr=10.244.0.0/16` 参数. 注意 Canal 只支持`amd64`.

```shell
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```

####  Flannel  

要使 `flannel`更好地运行 在执行 `kubeadm init` 命令式需要指定         `--pod-network-cidr=10.244.0.0/16` 参数 .

执行`sysctl net.bridge.bridge-nf-call-iptables=1`命令将`/proc/sys/net/bridge/bridge-nf-call-iptables` 的值设置为`1` 以便在iptables的链中允许IPv4的桥接网络数据通过。 其他的一些 CNI 也需要这样进行配置, 更多信息可以查看[这里](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
```
 `flannel` 支持 `amd64`, `arm`, `arm64` 和 `ppc64le`平台, 但是在 `flannel v0.11.0` 发布之前，需要使用下面的清单文件来支持所有的体系结构(architectures):  


```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/c5d10c8/Documentation/kube-flannel.yml
```

关于 `flannel`的更多信息可以查看, [the CoreOS flannel repository on GitHub
](https://github.com/coreos/flannel).

#### Kube-router  

通过运行`sysctl net.bridge.bridge-nf-call-iptables=1`将 `/proc/sys/net/bridge/bridge-nf-call-iptables` 设置为 `1` 以便iptables能够放行IPv4的桥接网络包。其他的一些 CNI 插件也需要这样进行配置, 更多信息可以查看[这里](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).

Kube-router 依赖 kube-controller-manager 为nodes节点分配 pod CIDR . 因此, 使用 `kubeadm init` 命令时需要加上 `--pod-network-cidr` 标志.

Kube-router 提供了基于服务代理的pod网络、网络策略和高性能IP虚拟服务器(IPVS)/Linux虚拟服务器(LVS).

使用kubeadm搭建基于Kube-router网络环境的 Kubernetes 集群  , 可以查看官方 [安装向导](https://github.com/cloudnativelabs/kube-router/blob/master/docs/kubeadm.md).

####  Romana  

通过运行`sysctl net.bridge.bridge-nf-call-iptables=1`将 `/proc/sys/net/bridge/bridge-nf-call-iptables` 设置为 `1` 以便iptables能够放行IPv4的桥接网络包。其他的一些 CNI 插件也需要这样进行配置, 更多信息可以查看[这里](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).

Romana 官方安装向导点击[这里](https://github.com/romana/romana/tree/master/containerize#using-kubeadm).

Romana 只支持 `amd64`.

```shell
kubectl apply -f https://raw.githubusercontent.com/romana/romana/master/containerize/specs/romana-kubeadm.yml
```

#### Weave Net  

通过运行`sysctl net.bridge.bridge-nf-call-iptables=1`将 `/proc/sys/net/bridge/bridge-nf-call-iptables` 设置为 `1` 以便iptables能够放行IPv4的桥接网络包。其他的一些 CNI 插件也需要这样进行配置, 更多信息可以查看[这里](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).

Weave Net 官方安装向导点击[这里](https://www.weave.works/docs/net/latest/kube-addon/).

Weave Net 支持 `amd64`, `arm`, `arm64` 和 `ppc64le` 并且不需要额外的操作.
Weave Net 默认设置为 hairpin 模式. 这允许pods 通过  Service IP  的地址直接访问他们自己.

```shell
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

#### JuniperContrail/TungstenFabric    

提供覆盖SDN解决方案，提供多云联网，混合云联网，同时覆盖-基础支持，网络政策执行，网络隔离，服务链接和灵活的负载平衡。

安装 JuniperContrail/TungstenFabric CNI 有多种灵活的方式.

点击这里快速开始: [TungstenFabric](https://tungstenfabric.github.io/website/)




一旦pod network 安装完成，就可以通过`kubectl get pods --all-namespaces` 这个命令的输出检查 CoreDNS pod 是否正在运行.如果CoreDNS pod正常运行，就可以持续加入新的节点了。

如果网络不正常或者 CoreDNS 不是运行态, 点击 [troubleshooting docs](troubleshooting-kubeadm)排查故障.

### Master 隔离

默认情况下，为了安全起见，您的集群不会调度主集群上的pod。如果想要调度主节点上的pod，例如开发环境中单机的kubernetes集群，可以运行下面的命令:

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

控制台输出像下面这样:

```
node "test-01" untainted
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```

This will remove the `node-role.kubernetes.io/master` taint from any nodes that
have it, including the master node, meaning that the scheduler will then be able
to schedule pods everywhere.

这个操作将把`node-role.kubernetes.io/master`这个标记从所有具有这个标记的节点上删除掉，包括master节点。这意味着调度器将能够将master上pod调度到任何地方。 

### 将节点加入集群   

这里所说的节点指的是运行容器和pod的节点.在每个机器上执行下面的命令，将这些节点加入到集群中:

* SSH 到节点主机
* 切换到 root (e.g. `sudo su -`)
* 运行加入到集群中的命令，这个命令在执行`kubeadm init`过程中有过提示 . For example:

``` bash
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

如果还没有获取到token, 可以在 master 节点上运行下面的命令获取:

``` bash
kubeadm token list
```

命令的输出类似下面这样:

``` console
TOKEN                    TTL  EXPIRES              USAGES           DESCRIPTION            EXTRA GROUPS
8ewj1p.9r9hcjoqgajrj4gi  23h  2018-06-12T02:51:28Z authentication,  The default bootstrap  system:
                                                   signing          token generated by     bootstrappers:
                                                                    'kubeadm init'.        kubeadm:
                                                                                           default-node-token
```

tokens 默认24小时之后失效。如果在token失效之后，还想将一个node节点加入到集群中去，可以在master节点上通过下面的命令生成一个新的token:

``` bash
kubeadm token create
```

输出类似于下面这样:

``` console
5didvk.d09sbcov8ph2amjw
```

如果不知道  `--discovery-token-ca-cert-hash` 的值是多少, 可以在master节点上执行下面的组合命令来获取:

``` bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

输出类似于下面这样:

``` console
8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78
```


> **Note:** 如果要按照 `<master-ip>:<master-port>` 这种格式指定IPv6地址, IPv6 地址必须被方括号括起来, 例如: `[fd00::101]:2073`.

输出像下面这样:

```
[preflight] Running pre-flight checks

... (log output of join workflow) ...

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

过一会儿，在master节点上执行`kubectl get nodes`命令，从输出的内容上就能看到节点已经加入到了集群中.

### (可选) 从master节点以外的机器控制集群

为了能够从master节点之外的计算机(例如笔记本)控制集群，我们需要从master节点将 administrator kubeconfig  配置文件复制到工作主机上(非master节点):

``` bash
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```


> **Note:** 上面的示例中假设了使用root来操作，这样是可行的。如果是其他用户，在copy `admin.conf` 配置文件时，请注意相应权限.

>  `admin.conf` 文件定义了集群的超级用户以及权限 .
这个文件应该谨慎使用. 对于普通用户则要求生成一个定义了白名单权限的唯一凭证. 可以使用 `kubeadm alpha phase kubeconfig user --client-name <CN>` 命令完成这项操作. 这个命令将打印一个 KubeConfig 文件到标准输出，我们需要将KubeConfig保存到文件中，并分发给其他的用户。然后使用`kubectl create (cluster)rolebinding`创建权限白名单.


### (可选) 将API服务器代理到本地主机


如果想要在集群之外链接到API Server 可以使用
`kubectl proxy`:

```bash
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
```

本地可以通过 `http://localhost:8001/api/v1` 这个地址访问API Server.

## 移除   

要撤销kubeadm所做的操作的话,在关闭节点之前需要将节点清空.

然后通知master节点:

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

然后在即将被移除掉的节点上，重置kubeadm的状态:

```bash
kubeadm reset
```

如果您希望重新开始，只需运行 `kubeadm init` 或者 `kubeadm join` 并加上合适的参数就可以了。

更多信息可以查看[`kubeadm reset command`]().

## 维护集群   

维护一个kubeadm集群的指令 (e.g. 升级,降级, etc.) 可以参考 [这里.]()



## 不足之处  

请注意: kubeadm 仍然还在开发过程中，有一些缺陷需要在这里说明.

1. 这里创建的集群只有一个master节点, 和一个单独的ETCD数据库.这意味着如果master节点宕机了, 集群将丢失所有数据，并需要从头开始重新创建. 给kubeadm 添加高可用
   (multiple etcd servers, multiple API servers, etc) 支持还在进展过程中.

   解决方案: 
   [备份etcd数据](https://coreos.com/etcd/docs/latest/admin_guide.html). master节点上被kubeadm 配置的etcd的数据路径是 `/var/lib/etcd` .

## 问题排查  

如果在使用 kubeadm遇到了困难, 可以点击 [问题排查手册](troubleshooting-kubeadm.html).来进行查阅.




