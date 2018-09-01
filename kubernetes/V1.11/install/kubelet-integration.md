## 使用kubeadm配置集群中的每个kubelet


kubeadm CLI工具的生命周期与Kubernetes节点代理是分离的。Kubernetes节点代理是运行在每个Master和Node节点上守护进程。kubelet会一直在后台运行，用户可以使用 kubeadm CLI 工具对Kubernetes 进行初始化或者升级操作。

既然kubelet是守护进程，它就需要被一些系统初始化进程(init
system)或者服务管理器(service manager)来进行维护。当使用deb或者rpm包来安装kubelet的时候，系统默认使用systemd来管理kubelet。 当然也可以使用其他的服务管理器，但是需要手动进行配置。

对于集群中涉及的所有kubelets，在某些配置配置内容上要保持一致。而其他得一些配置方面需要在每个kubelet所运行环境的基础上进行设置，以适应不同的情况机器的特性，如操作系统、存储和网络。

我们可以手动管理所有kubelets的配置。但是kubeadm现在提供了了一种 “MasterConfig” 的 API类型来集中管理kubelet的配置。

## 本章目录 

- Kubelet 配置模式
- 使用kubeadm配置 kubelets 
- The kubelet drop-in file for systemd
- Kubernetes的二进制文件和package内容

## Kubelet 配置模式

下面的内容介绍了使用kubeadm如何简化了 kubelet的配置模式。而不是人工管理每个节点的kubelet配置。

### 向每个 kubelet 推送集群配置

我们可以使用`kubelet init` 和 `kubelet join`命令来使kubelet应用默认值. 例如设置使用不同的 CRI 运行环境或者设置 serivice 使用的默认的子网等有趣的例子。

如果想让service 默认使用`10.96.0.0/12`这个子网 , 我们可以使用kubeadm时指定`--service-cidr` :

```bash
kubeadm init --service-cidr 10.96.0.0/12
```

services 的 虚拟ip(Virtual IPs) 也将从这个子网中进行分配. 同时需要为 kubelet指定`--cluster-dns` 参数来设置DNS. 这项设置要求集群中的每个master节点和node节点上的kubelet保持一致。kubelet 提供了一款版本化的，结构化的API对象。它能够 配置kubelet中大部分的参数，并把这些配置推送到集群中其他的kubelet中。这个API 对象被称作  **the kubelet's ComponentConfig**. 

ComponentConfig 允许用户指定自定义一些表达式来指定某些值。例如下面例子中指定  cluster DNS 地址的那一项。 这是yaml格式的配置文件:

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
clusterDNS:
- 10.96.0.10
```

可以点击
[API reference for the kubelet ComponentConfig](https://godoc.org/k8s.io/kubernetes/pkg/kubelet/apis/kubeletconfig#KubeletConfiguration)获取更多信息.

### 提供特定的配置细节

一些主机由于硬件，操作系统，网络环境，以及主机的一些特定参数的不同，需要指定kubelet配置。
例如下面的一些列子.

- DNS 解析的配置文件，可以通过为kubelet 指定 `--resolve-conf`参数来进行指定。 不同的操作系统之间会所不同，这取决你是否使用了`systemd-resolved` 类型的操作系统。如果DNS配置文件路径错误，DNS解析将会失败，kubelet配置也将会不正确。

- 除非使用了云服务，否则节点API 对象 `.metadata.name`默认使用主机的hostname. 如果需要将节点名称修改为与hostname不同的名称的话，可以通过指定`--hostname-override` 参数来进行修改。  

- 目前，kubelet无法自动检测CRI运行时使用的cgroup驱动程序, 但是  `--cgroup-driver` 参数的值，必须与CRI runtime使用的cgroup驱动程序保持一致，以便确定kubelet的健康状况。
 
- 根据集群使用的CRI runtime的不同，您可能需要为kubelet指定不同的标志.
  例如，使用docker的时候，需要指定`--network-plugin=cni`。但是如果使用了外部的运行时，需要指定 `--container-runtime=remote`  ,并且 CRI  endpoint 需要指定`--container-runtime-path-endpoint=<path>` .

您可以通过在服务管理器中配置单个kubelet的配置来指定这些标志,例如 systemd.

## 使用kubeadm 配置 kubelets

kubeadm的配置API 类型  `MasterConfiguration` 嵌入到了 kubelet ComponentConfig 的 `.kubeletConfiguration.baseConfig`  下面 。用户在写 `MasterConfiguration`
配置文件的时候可以使用这个配置为 集群中的所有 kubelets 设置基本配置.

### 使用 `kubeadm init` 时的过程

当我们执行`kubeadm init`的时候, `.kubeletConfiguration.baseConfig` 配置会被写入到磁盘上的  `/var/lib/kubelet/config.yaml` 文件中, 并且会上传到集群的 ConfigMap 中. 这里的ConfigMap 就是  `kubelet-config-1.X`,  `.X` 就是我们使用的 Kubernetes的小版本(例如当前版本1.11) . 集群中所有的kubelet的基本配置都会被写入 `/etc/kubernetes/kubelet.conf` 这个配置文件 . 这个配置文件指向了允许kubelete与API server 通信的客户端证书. 这样就能够满足
[向每个 kubelet 推送集群配置](#propagating-cluster-level-configuration-to-each-kubelet) 的需求.

为了满足[提供特定的配置细节](#providing-instance-specific-configuration-details),
kubeadm 写了一个环境变量文件 `/var/lib/kubelet/kubeadm-flags.env`,这个文件包含了在kubelet启动时，需要传递给他的一系列flag。这些flag类似于下面这样:

```bash
KUBELET_KUBEADM_ARGS="--flag1=value1 --flag2=value2 ..."
```

除了启动 kubelet 所需要的flags外, 这个文件还包含了一些如 cgroup driver 和是否需要使用不同的CRI runtime socket (`--cri-socket`) 的动态参数。

如果使用的是systemd 管理服务的话，在将这两个文件写入到磁盘之后，kubeadm会尝试运行下面的两个命令:

```bash
systemctl daemon-reload && systemctl restart kubelet
```

如果reload 和 restart 成功了话,  `kubeadm init` 过程才会正常进行.

### 使用 `kubeadm join` 时的过程

当我们运行 `kubeadm join` 命令的时候, kubeadm 会使用 Bootstrap Token 证书 来执行一个 TLS 引导, 这个TLS引导需要 `kubelet-config-1.X` ConfigMap 下载并写入 `/var/lib/kubelet/config.yaml`. 动态的环境配置文件与`kubeadm init`一样的方式生成。

接下来，`kubeadm` 会 运行下面两个命令将新的配置加载到kubelet中:

```bash
systemctl daemon-reload && systemctl restart kubelet
```

kubelet 加载了新得配置之后, kubeadm 会写
`/etc/kubernetes/bootstrap-kubelet.conf` 这个配置文件, 这个文件包含了一个 CA 证书和Bootstrap
Token. kubelet使用这些东西来进行 TLS 引导 并获得一个存储在`/etc/kubernetes/kubelet.conf`的唯一凭证。当配置文件被写完的时候, the kubelet
也就完成了 TLS Boo引导tstrap.

##  The kubelet drop-in file for systemd

通过 DEB 或者 RPM 包 安装的kubeadm ,配置文件会被写入到 `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`， 并且会被 systemd所使用.

```none
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
--kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating
the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably,
#the user should use the .NodeRegistration.KubeletExtraArgs object in the configuration files instead.
# KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

这文件指定了kubeadm为kubelet管理的所有文件的默认路径。.

- TLS Bootstrap 的配置文件`/etc/kubernetes/bootstrap-kubelet.conf`,但是他只有当
  `/etc/kubernetes/kubelet.conf` 不存在时才会使用.
- 具有kubelet唯一标识的配置文件 `/etc/kubernetes/kubelet.conf`.
- 包含 kubelet ComponentConfig 的文件 `/var/lib/kubelet/config.yaml`.
- 包含`KUBELET_KUBEADM_ARGS`的动态环境配置文件来源于 `/var/lib/kubelet/kubeadm-flags.env`.
- 包含用户指定的，能够覆盖`KUBELET_EXTRA_ARGS`这个参数的文件是 `/etc/default/kubelet` ( DEBs), 或者 `/etc/systconfig/kubelet` (RPMs). `KUBELET_EXTRA_ARGS`在flag链中位于最后一级，同时在配置冲突中拥有最高优先级。

## Kubernetes的二进制文件和package内容

与Kubernetes版本一起发布的DEB和RPM包像下面这样:

| Package name | Description |
|--------------|-------------|
| `kubeadm`    | Installs the `/usr/bin/kubeadm` CLI tool and [The kubelet drop-in file](#the-kubelet-drop-in-file-for-systemd) for the kubelet. |
| `kubelet`    | Installs the `/usr/bin/kubelet` binary. |
| `kubectl`    | Installs the `/usr/bin/kubectl` binary. |
| `kubernetes-cni` | Installs the official CNI binaries into the `/opt/cni/bin` directory. |
| `cri-tools` | Installs the `/usr/bin/crictl` binary from [https://github.com/kubernetes-incubator/cri-tools](https://github.com/kubernetes-incubator/cri-tools). |
