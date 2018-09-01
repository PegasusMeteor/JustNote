
## kubeadm故障排除


与任何程序一样，您可能在安装或运行kubeadm时遇到错误。  

本章列出了一些常见的故障场景，并提供了帮助理解和修复问题的步骤.  


如果你遇到的问题这里找不到, 可以参考下面的步骤:

- 如果你觉得你的问题是kubeadm的bug:
  - 跳转 [github.com/kubernetes/kubeadm](https://github.com/kubernetes/kubeadm/issues) 查看issues.
  - 如果没找到，根据issue 模板重新[open issues](https://github.com/kubernetes/kubeadm/issues/new).

- 如果不确定kubeadm是如何工作的,可以在Slack的 `#kubeadm` 主题下讨论也可以在StackOverflow 上提问. 然后标签选择`#kubernetes` 和 `#kubeadm` ，这样Google的技术人员才会给解答.


## 安装过程中没有找到 `ebtables` 或者其他的可执行程序

如果在运行 `kubeadm init` 命令时看到如下的警告

```sh
[preflight] WARNING: ebtables not found in system path
[preflight] WARNING: ethtool not found in system path
```

这个节点上可能缺失了 `ebtables`, `ethtool` 或者类似的可执行程序. 可以使用下面的命令来进行安装:

- For Ubuntu/Debian users, run `apt install ebtables ethtool`.
- For CentOS/Fedora users, run `yum install ebtables ethtool`.

## 安装过程总kubeadm 卡在了 waiting for control plane这个阶段  

I如果 `kubeadm init` 在输出了下面的内容之后就挂起了:

```sh
[apiclient] Created API client, waiting for the control plane to become ready
```

这可能导致了一系列的问题. 最有可能是:

- 网络链接错误 . 继续开始之前，确认主机已经具备完全联通的网络.
- kubelet默认使用的 cgroup driver 配置与Dokcer使用的不一致。
  检查一下系统的log文件 ( `/var/log/message`) 或者查看一下 `journalctl -u kubelet`的输出. 如果看到了下面的输出:

  ```shell
  error: failed to run Kubelet: failed to create kubelet:
  misconfiguration: kubelet cgroup driver: "systemd" is different from docker cgroup driver: "cgroupfs"
  ```

  这有两个方式来修改cgroup driver 问题:
  
 1. 按照下面的步骤重新安装docker，点击这里
  [here](install-kubeadm.html).
 1. 手动修改kubelet 的cgroup driver配置从而与 Docker保持一致, 可以参考[配置master节点上kubelet使用的 cgroup driver ](install-kubeadm.html).

- 控制台容器(control plane Docker containers) 故障或者挂起了. 可以运行 `docker ps` 查看容器，然后使用 `docker logs` 查看每个容器的log，以确定故障所在.

## 移除管理容器(managed containers)时kubeadm 卡住了   

如果Docker停止并没有删除任何kubernet管理的(Kubernetes-managed )容器，可能会发生以下情况:

```bash
sudo kubeadm reset
[preflight] Running pre-flight checks
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers
(block)
```

一个可能有用的解决方式是容器docker service 然后重新运行  `kubeadm reset`:

```bash
sudo systemctl restart docker.service
sudo kubeadm reset
```

检查docker的日志可能也很有用:

```sh
journalctl -ul docker
```

## Pods 处于  `RunContainerError`, `CrashLoopBackOff` 或者 `Error` 状态

 `kubeadm init` 命令执行成功之后，不应该有 pods 处于这些状态.

- 如果有 pods 在 `kubeadm init` 之后处于上面的某一种状态, 请在 kubeadm repo 上面open 一个issue.在我们部署了网络解决方案之前， `coredns` (或者 `kube-dns`) 应该是 `Pending` 状态 .
- 如果部署了网络之后，有pods处于 `RunContainerError`, `CrashLoopBackOff` 或者 `Error` 状态，并且`coredns` (or `kube-dns`) 没有变化，很有可能就是安装的pod网络出现了什么问题。 可能需要赋予它更多的RBAC(基于角色的权限访问控制Role-Based Access Control)权限 或者使用一个更新的版本. 并且请在 Pod Network providers 的官方open 一个issue,并持续跟踪。

## `coredns` (或者 `kube-dns`) 卡在了 `Pending` 状态

这是 **被期望的** ，也是设计的一部分. kubeadm 对网络模式是无感知的, 所以管理员在安装pod网络时需要进行选择. 在CoreDNS 被部署之前必须先安装 Pod Network. 因为在pod网络被安装之前是`Pending` 状态 .

## `HostPort` services 不起作用

 `HostPort` 和 `HostIP` 功能是否可用根据选择的不同的 Pod Network provider会有所不同. 可以联系不同的 Pod Network 作者查看
`HostPort` 和 `HostIP` 功能是否可用.

Calico, Canal, 和 Flannel CNI providers 是明确支持 HostPort的.

更多信息可以点击  [CNI portmap documentation](https://github.com/containernetworking/plugins/blob/master/plugins/meta/portmap/README.md)进行查看.

If your network provider does not support the portmap CNI plugin, you may need to use the [NodePort feature of
services](/docs/concepts/services-networking/service/#nodeport) or use `HostNetwork=true`.

## 通过 Service IP 不能访问 pods 

- 许多网络组件(add-ons) 至今不支持 [hairpin mode](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/#a-pod-cannot-reach-itself-via-service-ip) (允许pods通过他们的Service IP 访问他们自己). 这里有一个issue 
  [CNI](https://github.com/containernetworking/cni/issues/476). 可以联系 network 组件(add-on) 提供者 获取 他们对 hairpin mode 支持的最新进展 .

- 如果正在使用 VirtualBox (直接或者通过Vagrant), 你需要确认`hostname -i`命令能够返回可路由的IP地址. 默认情况下，第一个接口连接到不可路由的仅主机网络.一个变通的方法是修改`/etc/hosts`, 例如[Vagrantfile](https://github.com/errordeveloper/k8s-playground/blob/22dd39dfc06111235620e6c4404a96ae146f26fd/Vagrantfile#L11).

## TLS 证书错误

下面的错误表示证书可能不匹配.

```none
# kubectl get pods
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

- 确认 `$HOME/.kube/config` 文件中包含了有效的证书, 有必要的话重新生成一个.  kubeconfig 文件中的证书是 base64 编码的.  `base64 -d` 命令可以用来对证书解码 ， `openssl x509 -text -noout` 可以用来查看证书信息.
- 另一个变通的方法是为管理员用户重写 `kubeconfig` 配置文件:

  ```sh
  mv  $HOME/.kube $HOME/.kube.bak
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

## 在 Vagrant 中使用 flannel 作为pod network时的默认NIC

下面的错误表明 pod network 中有错误:

```sh
Error from server (NotFound): the server could not find the requested resource
```

- 如果在Vagrant 中使用 flannel 作为 pod network ,必须为flannel 指定default interface name.

  Vagrant 默认对所有的虚拟机指定两个网络接口. 所有主机的第一个网卡都指定了 `10.0.2.15` IP地址, 用来处理经过NAT转换的外部流量.

  flannel 默认使用主机上的第一个网卡，这就可能会导致出现问题。这会让所有的主机认为，他们具有相同的ip地址。 为了防止这个发生，给flannel 加上 `--iface eth1` 参数，这样第二块网卡就被选中了。

## Non-public IP used for containers

在一些场景中 `kubectl logs` 和 `kubectl run` 命令可能会返回下面这样一些错误:

```sh
Error from server: Get https://10.19.0.41:10250/containerLogs/default/mysql-ddc65b868-glc5m/mysql: dial tcp 10.19.0.41:10250: getsockopt: no route to host
```

- 这很有可能是由于 Kubernetes 使用了看起来是同一个子网，但是互相之间却不能通信的ip地址。
- Digital Ocean 分配了一个 public IP 给 `eth0` ，因为他们的 浮动IP特性，还会分配一个私有的IP来内部使用。 现在 `kubelet` 会选择后者作为节点的 `InternalIP` 而不是public IP.

  使用 `ip addr show` 来检查一下这个场景。 不要使用 `ifconfig` 命令来操作，因为 `ifconfig` 不会显示别名ip地址。. 另外， Digital Ocean 指定的 API 终端 允许查询droplet的内部IP:

  ```sh
  curl http://169.254.169.254/metadata/v1/interfaces/public/0/anchor_ipv4/address
  ```

  解决办法就是使用 `--node-ip` 告诉 `kubelet` 使用哪个ip . 当使用 Digital Ocean 时, 可能是 public 的那个 (指向了 `eth0`) , 也有可能是private 的那个 (指向了 `eth1`) 如果你想用私有网络的话.  [KubeletExtraArgs section of the MasterConfiguration file](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/apis/kubeadm/v1alpha2/types.go#L147) 就可以这样来用.

  然后重启 `kubelet`:

  ```sh
  systemctl daemon-reload
  systemctl restart kubelet
  ```

## Services with externalTrafficPolicy=Local are not reachable

在那些使用了`--hostname-override` 选项重写了kubelet的hostname 的节点上 kube-proxy 会默认将 127.0.0.1 设为节点IP, 结果会因为 Services 被配置成了`externalTrafficPolicy=Local` 而拒绝链接. 可以通过`kubectl -n kube-system logs <kube-proxy pod name>`确认是否存在这种状况:

```sh
W0507 22:33:10.372369       1 server.go:586] Failed to retrieve node info: nodes "ip-10-0-23-78" not found
W0507 22:33:10.372474       1 proxier.go:463] invalid nodeIP, initializing kube-proxy with 127.0.0.1 as nodeIP
```

解决这个问题的方法是按照以下方式修改kube-proxy DaemonSet:

```sh
kubectl -n kube-system patch --type json daemonset kube-proxy -p "$(cat <<'EOF'
[
    {
        "op": "add",
        "path": "/spec/template/spec/containers/0/env",
        "value": [
            {
                "name": "NODE_NAME",
                "valueFrom": {
                    "fieldRef": {
                        "apiVersion": "v1",
                        "fieldPath": "spec.nodeName"
                    }
                }
            }
        ]
    },
    {
        "op": "add",
        "path": "/spec/template/spec/containers/0/command/-",
        "value": "--hostname-override=${NODE_NAME}"
    }
]
EOF
)"

```



