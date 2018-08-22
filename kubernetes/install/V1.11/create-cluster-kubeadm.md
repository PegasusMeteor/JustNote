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


- 开始之前
- 目标
- 安装过程
- Tear down
- Maintaining a cluster
- Explore other add-ons
- What’s next
- Feedback
- Version skew policy
- kubeadm works on multiple platforms
- Limitations
- Troubleshooting


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

See ["Installing kubeadm"](install-kubeadm).

> **注意:** 如果已经安装了 kubeadm, 运行 `apt-get update &&
apt-get upgrade` 或者 `yum update` 去获取最新版本的 kubeadm.

在升级的过程中，kubelet会陷入死循环，每隔几秒钟重启一下，直到kubeadm通知它要做什么。死循环是正常并且有必要的。
初始化了 master节点之后, kubelet 会回复正常.  


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

If you join a node with a different architecture to your cluster, create a separate
Deployment or DaemonSet for `kube-proxy` and `kube-dns` on the node. This is because the Docker images for these
components do not currently support multi-architecture.
如果要将一个不同体系结构(architecture)的node假如到集群中，在这个节点上需要单独创建`kube-proxy` 和 `kube-dns`。因为这些组件的Docker 镜像不支持 多体系结构(multi-architecture)。


`kubeadm init` 命令首先会进行一系列的检测以确保这台机器适合运行kubernetes. 这些检测或输出warnings并在出现errors的时候停止掉. 然后 `kubeadm init` 命令会下载集群控制台(cluster control plane)需要的各种组件. 需要几分钟的时间. 输出应该会像下面这个样子:

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
为了非root用户更好的使用kubectl，可以执行下面的命令，这些内容 也是`kubeadm init` 执行过程中会提示的内容:

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

### 安装 POD 网络组件 {#pod-network}


> **注意:** 本节包含关于安装和部署顺序的重要信息。在继续之前请仔细阅读。


因为pod要求互相之间能够正常通信，所以必须安装网络组件，


**在发布应用程序之前，必须先将网络调通. 并且,  在网络没有安装好之前也不能启动CoreDNS.
kubeadm 只支持基于网络的容器网络接口(Container Network Interface (CNI)) (不支持 kubenet).**

有几个项目使用CNI提供Kubernetes pod网络，其中一些还支持网络策略。
- [CNI v0.6.0](https://github.com/containernetworking/cni/releases/tag/v0.6.0) 开始支持IPv6.
- [CNI bridge](https://github.com/containernetworking/plugins/blob/master/plugins/main/bridge/README.md) 和 [local-ipam](https://github.com/containernetworking/plugins/blob/master/plugins/ipam/host-local/README.md) 是 Kubernetes  1.9 版本中唯一支持IPv6的组件.

Note that kubeadm sets up a more secure cluster by default and enforces use of [RBAC](/docs/reference/access-authn-authz/rbac/).
Make sure that your network manifest supports RBAC.

You can install a pod network add-on with the following command:

```bash
kubectl apply -f <add-on.yaml>
```

每个集群中可以只安装一个 pod network.



根据下面不同的网络提供商选择不同的安装方式。


**Calico**  

可以点击 [Quickstart for Calico on Kubernetes](https://docs.projectcalico.org/latest/getting-started/kubernetes/), [Installing Calico for policy and networking](https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/calico),获取更多信息.

为了使网络策略正确工作, 在运行`kubeadm init`命令时需要指定 `--pod-network-cidr=192.168.0.0/16` . 注意 Calico 只支持 `amd64` 平台.

```shell
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

**Canal**
Canal 使用Calico做网络策略，Flannel来进行网络转发。点击后面链接查看 Calico 文档 [official getting started guide](https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/flannel).

要使Canal更好地运行,在运行 `kubeadm init` 命令时需要指定 `--pod-network-cidr=10.244.0.0/16` 参数. 注意 Canal 只支持`amd64`.

```shell
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```

**Flannel**

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

**Kube-router**
通过运行`sysctl net.bridge.bridge-nf-call-iptables=1`将 `/proc/sys/net/bridge/bridge-nf-call-iptables` 设置为 `1` 以便iptables能够放行IPv4的桥接网络包。其他的一些 CNI 插件也需要这样进行配置, 更多信息可以查看[这里](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).

Kube-router 依赖 kube-controller-manager 为nodes节点分配 pod CIDR . 因此, 使用 `kubeadm init` 命令时需要加上 `--pod-network-cidr` 标志.

Kube-router 提供了基于服务代理的pod网络、网络策略和高性能IP虚拟服务器(IPVS)/Linux虚拟服务器(LVS).

使用kubeadm搭建基于Kube-router网络环境的 Kubernetes 集群  , 可以查看官方 [安装向导](https://github.com/cloudnativelabs/kube-router/blob/master/docs/kubeadm.md).

**Romana**

通过运行`sysctl net.bridge.bridge-nf-call-iptables=1`将 `/proc/sys/net/bridge/bridge-nf-call-iptables` 设置为 `1` 以便iptables能够放行IPv4的桥接网络包。其他的一些 CNI 插件也需要这样进行配置, 更多信息可以查看[这里](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).

Romana 官方安装向导点击[这里](https://github.com/romana/romana/tree/master/containerize#using-kubeadm).

Romana 只支持 `amd64`.

```shell
kubectl apply -f https://raw.githubusercontent.com/romana/romana/master/containerize/specs/romana-kubeadm.yml
```

**Weave Net**

通过运行`sysctl net.bridge.bridge-nf-call-iptables=1`将 `/proc/sys/net/bridge/bridge-nf-call-iptables` 设置为 `1` 以便iptables能够放行IPv4的桥接网络包。其他的一些 CNI 插件也需要这样进行配置, 更多信息可以查看[这里](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).

Weave Net 官方安装向导点击[这里](https://www.weave.works/docs/net/latest/kube-addon/).

Weave Net 支持 `amd64`, `arm`, `arm64` 和 `ppc64le` 并且不需要额外的操作.
Weave Net 默认设置为 hairpin 模式. 这允许pods 通过  Service IP  的地址直接访问他们自己.

```shell
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

**JuniperContrail/TungstenFabric**  

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

### 将节点加入集群 {#join-nodes}

The nodes are where your workloads (containers and pods, etc) run. To add new nodes to your cluster do the following for each machine:

* SSH to the machine
* Become root (e.g. `sudo su -`)
* Run the command that was output by `kubeadm init`. For example:

``` bash
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

If you do not have the token, you can get it by running the following command on the master node:

``` bash
kubeadm token list
```

The output is similar to this:

``` console
TOKEN                    TTL  EXPIRES              USAGES           DESCRIPTION            EXTRA GROUPS
8ewj1p.9r9hcjoqgajrj4gi  23h  2018-06-12T02:51:28Z authentication,  The default bootstrap  system:
                                                   signing          token generated by     bootstrappers:
                                                                    'kubeadm init'.        kubeadm:
                                                                                           default-node-token
```

By default, tokens expire after 24 hours. If you are joining a node to the cluster after the current token has expired,
you can create a new token by running the following command on the master node:

``` bash
kubeadm token create
```

The output is similar to this:

``` console
5didvk.d09sbcov8ph2amjw
```

If you don't have the value of `--discovery-token-ca-cert-hash`, you can get it by running the following command chain on the master node:

``` bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

The output is similar to this:

``` console
8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78
```

{{< note >}}
**Note:** To specify an IPv6 tuple for `<master-ip>:<master-port>`, IPv6 address must be enclosed in square brackets, for example: `[fd00::101]:2073`.
{{< /note >}}

The output should look something like:

```
[preflight] Running pre-flight checks

... (log output of join workflow) ...

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

A few seconds later, you should notice this node in the output from `kubectl get
nodes` when run on the master.

### (Optional) Controlling your cluster from machines other than the master

In order to get a kubectl on some other computer (e.g. laptop) to talk to your
cluster, you need to copy the administrator kubeconfig file from your master
to your workstation like this:

``` bash
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```

{{< note >}}
**Note:** The example above assumes SSH access is enabled for root. If that is not the
case, you can copy the `admin.conf` file to be accessible by some other user
and `scp` using that other user instead.

The `admin.conf` file gives the user _superuser_ privileges over the cluster.
This file should be used sparingly. For normal users, it's recommended to
generate an unique credential to which you whitelist privileges. You can do
this with the `kubeadm alpha phase kubeconfig user --client-name <CN>`
command. That command will print out a KubeConfig file to STDOUT which you
should save to a file and distribute to your user. After that, whitelist
privileges by using `kubectl create (cluster)rolebinding`.
{{< /note >}}

### (Optional) Proxying API Server to localhost

If you want to connect to the API Server from outside the cluster you can use
`kubectl proxy`:

```bash
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
```

You can now access the API Server locally at `http://localhost:8001/api/v1`

## Tear down {#tear-down}

To undo what kubeadm did, you should first [drain the
node](/docs/reference/generated/kubectl/kubectl-commands#drain) and make
sure that the node is empty before shutting it down.

Talking to the master with the appropriate credentials, run:

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

Then, on the node being removed, reset all kubeadm installed state:

```bash
kubeadm reset
```

If you wish to start over simply run `kubeadm init` or `kubeadm join` with the
appropriate arguments.

More options and information about the
[`kubeadm reset command`](/docs/reference/setup-tools/kubeadm/kubeadm-reset/).

## Maintaining a cluster {#lifecycle}

Instructions for maintaining kubeadm clusters (e.g. upgrades,downgrades, etc.) can be found [here.](/docs/tasks/administer-cluster/kubeadm)

## Explore other add-ons {#other-addons}

See the [list of add-ons](/docs/concepts/cluster-administration/addons/) to explore other add-ons,
including tools for logging, monitoring, network policy, visualization &amp;
control of your Kubernetes cluster.

## What's next {#whats-next}

* Verify that your cluster is running properly with [Sonobuoy](https://github.com/heptio/sonobuoy)
* Learn about kubeadm's advanced usage in the [kubeadm reference documentation](/docs/reference/setup-tools/kubeadm/kubeadm)
* Learn more about Kubernetes [concepts](/docs/concepts/) and [`kubectl`](/docs/user-guide/kubectl-overview/).
* Configure log rotation. You can use **logrotate** for that. When using Docker, you can specify log rotation options for Docker daemon, for example `--log-driver=json-file --log-opt=max-size=10m --log-opt=max-file=5`. See [Configure and troubleshoot the Docker daemon](https://docs.docker.com/engine/admin/) for more details.

## Feedback {#feedback}

* For bugs, visit [kubeadm Github issue tracker](https://github.com/kubernetes/kubeadm/issues)
* For support, visit kubeadm Slack Channel:
  [#kubeadm](https://kubernetes.slack.com/messages/kubeadm/)
* General SIG Cluster Lifecycle Development Slack Channel:
  [#sig-cluster-lifecycle](https://kubernetes.slack.com/messages/sig-cluster-lifecycle/)
* SIG Cluster Lifecycle [SIG information](#TODO)
* SIG Cluster Lifecycle Mailing List:
  [kubernetes-sig-cluster-lifecycle](https://groups.google.com/forum/#!forum/kubernetes-sig-cluster-lifecycle)

## Version skew policy {#version-skew-policy}

The kubeadm CLI tool of version vX.Y may deploy clusters with a control plane of version vX.Y or vX.(Y-1).
kubeadm CLI vX.Y can also upgrade an existing kubeadm-created cluster of version vX.(Y-1).

Due to that we can't see into the future, kubeadm CLI vX.Y may or may not be able to deploy vX.(Y+1) clusters.

Example: kubeadm v1.8 can deploy both v1.7 and v1.8 clusters and upgrade v1.7 kubeadm-created clusters to
v1.8.

Please also check our [installation guide](/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)
for more information on the version skew between kubelets and the control plane.

## kubeadm works on multiple platforms {#multi-platform}

kubeadm deb/rpm packages and binaries are built for amd64, arm (32-bit), arm64, ppc64le, and s390x
following the [multi-platform
proposal](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/multi-platform.md).

Only some of the network providers offer solutions for all platforms. Please consult the list of
network providers above or the documentation from each provider to figure out whether the provider
supports your chosen platform.

## Limitations {#limitations}

Please note: kubeadm is a work in progress and these limitations will be
addressed in due course.

1. The cluster created here has a single master, with a single etcd database
   running on it. This means that if the master fails, your cluster may lose
   data and may need to be recreated from scratch. Adding HA support
   (multiple etcd servers, multiple API servers, etc) to kubeadm is
   still a work-in-progress.

   Workaround: regularly
   [back up etcd](https://coreos.com/etcd/docs/latest/admin_guide.html). The
   etcd data directory configured by kubeadm is at `/var/lib/etcd` on the master.

## Troubleshooting {#troubleshooting}

If you are running into difficulties with kubeadm, please consult our [troubleshooting docs](/docs/setup/independent/troubleshooting-kubeadm/).




