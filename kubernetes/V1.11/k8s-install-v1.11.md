

## kubernetes V1.11 版本 安装实践

### 实验环境介绍
|主机名称|主机角色|IP地址|
|:---:|:---:|:---:|  
|k8s-master|master|192.168.0.39|
|k8s-node1|node|192.168.0.40|  
|k8s-node2|node|192.168.0.41|
|k8s-node3|node|192.168.0.42|   

实验环境组件结构图如下所示。   

![实验环境架构图](http://www.selinux.tech/k8s/k8s-install-v1.11/k8s-env-structure.jpg) 
  

实验环境的网络结构如下图所示。至于为什么使用这个网段，后面的过程中会有详细的介绍。  

![实验网络图](http://www.selinux.tech/k8s/k8s-install-v1.11/k8s-env-network.jpg) 


### 实验环境准备
1.  关闭防火墙或者iptables  

    ```shell
    systemctl disable firewalld && systemctl stop firewalld  

    systemctl diables iptables && systemctl stop iptables
    ```


1.  关闭selinux  
在 `/etc/selinux/config`文件修改下列配置  

    ```shell
    SELINUX=disabled
    ```

1. 确保时间同步

    生产中一定要有时间服务器,安装时间服务器ntp。详细过程可以自行查阅。如果有必要的话，设置一下时区。  
    ```shell
    timedatectl set-timezone Asia/Shanghai & systemctl restart chronyd.service
    ```  

1. 设置所有的主机可以通过主机名进行访问  
    编辑 `/etc/hosts`文件，添加下面的内容
    ``` shell
    192.168.0.39 k8s-master
    192.168.0.40 k8s-node1
    192.168.0.41 k8s-node2
    192.168.0.42 k8s-node3

    ```  

    否则的话，在对master 进行 `kubeadm init `命令的时候，会出现如下的警告信息。 
    ```shell
    [WARNING Hostname]: hostname "k8s-master" could not be reached
    [WARNING Hostname]: hostname "k8s-master" lookup k8s-master on 192.168.1.1:53: no such host

    ```   

1. 设置系统相关参数  
    这些参数是下面实验过程中某些环节必须的。只不过我们将其提前在这里描述了而已。如果想要搞明白是在哪个环节需要做这些事情的话，可以先不用设置，后面遇到了再参考这里进行解决就可以了。  

    ```shell
    sysctl net.bridge.bridge-nf-call-iptables=1
    sysctl net.ipv4.ip_forward=1
    sysctl -p

    ```


### 安装

由于某些不可描述的原因，我们在安装各项组件的时候，可能会遇到网络的问题。所以建议在安装各项组件尽量选择国内的网络。

#### 配置安装需要的yum源  

安装过程中需要解决一些依赖问题，所以在安装之前，最好将CentOS的 base源，以及epel源和extra源都配置好，避免安装过程中，出现了错误导致安装中止。

**1、kubernetes yum 源**  

kubernetes的安装可以参考官方的文档来进行操作。  
官方文档地址[https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl](https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)。  
如果阅读困难的话，可以参考我的版本。[https://xiaoshuaigege.gitbooks.io/justnote/content/kubernetes/install/V1.11/_index.html](https://xiaoshuaigege.gitbooks.io/justnote/content/kubernetes/install/V1.11/_index.html)   

鉴于前面网络的原因，我们选择使用阿里云的yum源，地址 [https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/](https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/)  

```shell
[name]
name=Kubernetes rpm repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
```    

实验过程中使用的版本是   
```shell
~]# yum info kubelet
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Available Packages
Name        : kubelet
Arch        : x86_64
Version     : 1.11.2
Release     : 0
Size        : 18 M
Repo        : name
Summary     : Container cluster management
URL         : https://kubernetes.io
License     : ASL 2.0
Description : The node agent of Kubernetes, the container cluster manager.
```  

**2、配置Docker yum源**  

这里实验中安装的是Docker-ce的版本。国内的开源镜像站同样提供了比较便捷的安装方式。我们同样使用阿里云的yum源。镜像地址[https://mirrors.aliyun.com/docker-ce/linux/centos/](https://mirrors.aliyun.com/docker-ce/linux/centos/)  

```shell
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
```  

实验过程中使用的docker版本是   

```
Client:
 Version:           18.06.0-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        0ffa825
 Built:             Wed Jul 18 19:08:18 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.0-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       0ffa825
  Built:            Wed Jul 18 19:10:42 2018
  OS/Arch:          linux/amd64
  Experimental:     false

```  

### 在master节点上安装Docker、kubeadm、kubelet、kubectl

```shell
yum install  kubeadm kubelet kubectl docker-ce
```

### 在node节点上安装Docker、kubelet、kubectl

```shell
yum install kubeadm kubelet docker-ce

```

### Docker 加速 

为了后面实验能够顺利进行，这里对Docker 配置一下加速。使用的是阿里云的加速配置,阿里云的加速，对每个用户而言是不一样的。   

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://vbbcsxj5.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker
```


### 对master进行初始化 

在以往的旧版本中，需要修改 kubeadm 的cgroup-driver与安装的docker所使用的一致。  

最新版本的kubenetes 已经发生了变化，会自动根据docker中运行的cgroup-driver进行配置变更，这个在官方网站上已经介绍了。

官方建议禁用 swap，但也可以使用`--ignore-preflight-errors` 这个参数忽略掉这个error。   

```shell

kubeadm init --help 

kubeadm init  --kubernetes-version=v1.11.2 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap   

```
其中之所以指定了`--kubernetes-version=v1.11.2` 这个版本参数，是因为如果不指定的话，在进行初始化的时候会访问google的网络，导致出现下面的错误，所以这里我们进行了手动指定版本。   

```shell
unable to get URL "https://dl.k8s.io/release/stable-1.11.txt": Get https://storage.googleapis.com/kubernetes-release/release/stable-1.11.txt: dial tcp 216.58.220.208:443: i/o timeout
```

`--pod-network-cidr=10.244.0.0/16` 这个参数的设置是因为我们选择了pod的网络组件为flannel.flannel 默认的网络就是这个网段。 `--service-cidr=10.96.0.0/12` 参数的作用是让service 默认使用这个子网网段。   


这个过程可能会出现下面的错误。

```shell
[root@k8s-master ~]# kubeadm init  --kubernetes-version=v1.11.2 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap 
[init] using Kubernetes version: v1.11.2
[preflight] running pre-flight checks
I0829 22:23:08.443616    3209 kernel_validator.go:81] Validating kernel version
I0829 22:23:08.443829    3209 kernel_validator.go:96] Validating kernel config
        [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 18.06.1-ce. Max validated version: 17.03
[preflight/images] Pulling images required for setting up a Kubernetes cluster
[preflight/images] This might take a minute or two, depending on the speed of your internet connection
[preflight/images] You can also perform this action in beforehand using 'kubeadm config images pull'
[preflight] Some fatal errors occurred:
        [ERROR ImagePull]: failed to pull image [k8s.gcr.io/kube-apiserver-amd64:v1.11.2]: exit status 1
        [ERROR ImagePull]: failed to pull image [k8s.gcr.io/kube-controller-manager-amd64:v1.11.2]: exit status 1
        [ERROR ImagePull]: failed to pull image [k8s.gcr.io/kube-scheduler-amd64:v1.11.2]: exit status 1
        [ERROR ImagePull]: failed to pull image [k8s.gcr.io/kube-proxy-amd64:v1.11.2]: exit status 1
        [ERROR ImagePull]: failed to pull image [k8s.gcr.io/pause:3.1]: exit status 1
        [ERROR ImagePull]: failed to pull image [k8s.gcr.io/etcd-amd64:3.2.18]: exit status 1
        [ERROR ImagePull]: failed to pull image [k8s.gcr.io/coredns:1.1.3]: exit status 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
```

这是由于国内网络环境，不能访问国外的地址导致需要的镜像下载不了。  

这里的解决方式有两种：

**第一种是去阿里云下载相关镜像，然后再使用docker tag 修改**。


阿里云镜像地址 
[https://dev.aliyun.com/list.html?namePrefix=ks](https://dev.aliyun.com/list.html?namePrefix=ks)


注意下载的版本，应该与要求的版本一致
```shell
docker pull registry.cn-beijing.aliyuncs.com/acs/kube-apiserver-amd64:v1.11.2
docker pull registry.cn-beijing.aliyuncs.com/acs/kube-controller-manager-amd64:v1.11.2
docker pull registry.cn-beijing.aliyuncs.com/acs/kube-scheduler-amd64:v1.11.2
docker pull registry.cn-beijing.aliyuncs.com/acs/kube-proxy-amd64:v1.11.2
docker pull registry.cn-beijing.aliyuncs.com/acs/pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd-amd64:3.2.18
docker pull registry.cn-beijing.aliyuncs.com/acs/coredns:1.1.3

```

修改docker image tag  

```
docker tag  [image_id]  k8s.gcr.io/kube-apiserver-amd64:v1.11.2
docker tag  [image_id]  k8s.gcr.io/kube-controller-manager-amd64:v1.11.2
docker tag  [image_id]  k8s.gcr.io/kube-scheduler-amd64:v1.11.2
docker tag  [image_id]  k8s.gcr.io/kube-proxy-amd64:v1.11.2
docker tag  [image_id]  k8s.gcr.io/pause:3.1
docker tag  [image_id]  k8s.gcr.io/etcd-amd64:3.2.18
docker tag  [image_id]  k8s.gcr.io/coredns:1.1.3

```  

将旧的tag 删除掉  

```shell
docker rmi registry.cn-beijing.aliyuncs.com/acs/kube-apiserver-amd64:v1.11.2
docker rmi registry.cn-beijing.aliyuncs.com/acs/kube-controller-manager-amd64:v1.11.2
docker rmi registry.cn-beijing.aliyuncs.com/acs/kube-scheduler-amd64:v1.11.2
docker rmi registry.cn-beijing.aliyuncs.com/acs/kube-proxy-amd64:v1.11.2
docker rmi registry.cn-beijing.aliyuncs.com/acs/pause:3.1
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/etcd-amd64:3.2.18
docker rmi registry.cn-beijing.aliyuncs.com/acs/coredns:1.1.3

```
这样的话，剩下的docker image 就是系统初始化需要的image 了。  

```shell
[root@k8s-master ~]# docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-apiserver-amd64            v1.11.2             821507941e9c        3 weeks ago         187MB
k8s.gcr.io/kube-controller-manager-amd64   v1.11.2             38521457c799        3 weeks ago         155MB
k8s.gcr.io/kube-proxy-amd64                v1.11.2             46a3cd725628        3 weeks ago         97.8MB
k8s.gcr.io/kube-scheduler-amd64            v1.11.2             37a1403e6c1a        3 weeks ago         56.8MB
k8s.gcr.io/coredns                         1.1.3               b3b94275d97c        3 months ago        45.6MB
k8s.gcr.io/etcd-amd64                      3.2.18              b8df3b177be2        4 months ago        219MB
k8s.gcr.io/pause                           3.1                 da86e6ba6ca1        8 months ago        742kB
```

**第二种是修改docker的配置，使用http代理**  

第一种方式是可以解决掉镜像下载的问题的，但是，如果集群中有众多的节点，而每个节点都需要这样进行操作的话，免不了耗费大量的时间，因此，推荐使用第二种http代理的方式。 

**注意**,下面代码中的代理，在实际使用时，需要替换成自己的代理，笔者也只是在学习的过程中借鉴了别人的代理，目前该代理已经不可用(2018.10.21补充)

编辑docker的服务文件`/usr/lib/systemd/system/docker.service`，在其中 Service 部分 加入下面这样两个`Environment`环境变量，然后重启docker.  

```shell
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
Environment="HTTPS_PROXY=http://www.ik8s.io:10080"
Environment="NO_PROXY=127.0.0.1/8,192.168.0.0/16"
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
```  

因为修改了system文件，需要重新reload服务，并重启docker.

```shell
systemctl daemon-reload
systemctl restart docker
```

这样，就能够正常的从网络中下载我们所需要的镜像文件了。  



上面一系列的问题解决之后，继续运行 init 过程 会出现下面的结果 

```shell
[root@k8s-master ~]# kubeadm init  --kubernetes-version=v1.11.2 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap  
[init] using Kubernetes version: v1.11.2
[preflight] running pre-flight checks
I0829 22:40:10.083792    3785 kernel_validator.go:81] Validating kernel version
I0829 22:40:10.084038    3785 kernel_validator.go:96] Validating kernel config
        [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 18.06.1-ce. Max validated version: 17.03
[preflight/images] Pulling images required for setting up a Kubernetes cluster
[preflight/images] This might take a minute or two, depending on the speed of your internet connection
[preflight/images] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[preflight] Activating the kubelet service
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.0.39]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Generated etcd/ca certificate and key.
[certificates] Generated etcd/server certificate and key.
[certificates] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [127.0.0.1 ::1]
[certificates] Generated etcd/peer certificate and key.
[certificates] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [192.168.0.39 127.0.0.1 ::1]
[certificates] Generated etcd/healthcheck-client certificate and key.
[certificates] Generated apiserver-etcd-client certificate and key.
[certificates] valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[controlplane] wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests" 
[init] this might take a minute or longer if the control plane images have to be pulled
[apiclient] All control plane components are healthy after 49.508501 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.11" in namespace kube-system with the configuration for the kubelets in the cluster
[markmaster] Marking the node k8s-master as master by adding the label "node-role.kubernetes.io/master=''"
[markmaster] Marking the node k8s-master as master by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "k8s-master" as an annotation
[bootstraptoken] using token: fw3j8a.ynw60celvzihylou
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
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

  kubeadm join 192.168.0.39:6443 --token fw3j8a.ynw60celvzihylou --discovery-token-ca-cert-hash sha256:700ce85cde3a8cae39c4c957269090f20062ed5b397d214f261b33ef68404145

```
至此 kubernetes 集群的 master 已经初始化完成

在 `init` 命令执行成功之后，命令行会提示接下来应该怎样操作。  

### 创建配置文件  

为了实验方便，我们直接直接使用root来进行操作。

```shell
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```    

执行此操作之后，就可以使用 `kubectl` 命令了.例如,查看master节点上的组件状态：

```shell
[root@k8s-master ~]# kubectl get cs                
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok                   
scheduler            Healthy   ok                   
etcd-0               Healthy   {"health": "true"} 
```    

查看集群中现有节点以及其状态：

```shell
[root@k8s-master ~]# kubectl  get nodes
NAME         STATUS     ROLES     AGE       VERSION
k8s-master   NotReady   master    22h       v1.11.2  

```

集群中只有一个节点是因为其他的节点还没有join到集群中去。 `NotReady` 是因为我们的 `Pod Network` 还没有安装。也就是flannel 还没有安装。  


### 安装flannel  
flannel 本身也是一个开源项目。可以到github的站点上，去查看如何部署flannel。[https://github.com/coreos/flannel](https://github.com/coreos/flannel)    

执行下面的命令  

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 
```  

当输出下面的命令的内容的时候，并不代表已经安装成功，相反我们需要等待flannel 安装完成。  

```shell
[root@k8s-master ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created
```    

过一段时间之后，可以通过 ` docker image ls `,`kubectl get pods  -n kube-system -o wide` 命令查看是否 flannel 已经安装成功。  
```shell  
[root@k8s-master ~]# docker image ls
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-apiserver-amd64            v1.11.2             821507941e9c        3 weeks ago         187MB
k8s.gcr.io/kube-controller-manager-amd64   v1.11.2             38521457c799        3 weeks ago         155MB
k8s.gcr.io/kube-proxy-amd64                v1.11.2             46a3cd725628        3 weeks ago         97.8MB
k8s.gcr.io/kube-scheduler-amd64            v1.11.2             37a1403e6c1a        3 weeks ago         56.8MB
k8s.gcr.io/coredns                         1.1.3               b3b94275d97c        3 months ago        45.6MB
k8s.gcr.io/etcd-amd64                      3.2.18              b8df3b177be2        4 months ago        219MB
quay.io/coreos/flannel                     v0.10.0-amd64       f0fad859c909        7 months ago        44.6MB
k8s.gcr.io/pause                           3.1                 da86e6ba6ca1        8 months ago        742kB  



[root@k8s-master ~]# kubectl get pods  -n kube-system -o wide
NAME                                 READY     STATUS    RESTARTS   AGE       IP             NODE         NOMINATED NODE
coredns-78fcdf6894-khlkd             1/1       Running   0          23h       10.244.0.3     k8s-master   <none>
coredns-78fcdf6894-qqrnc             1/1       Running   0          23h       10.244.0.2     k8s-master   <none>
etcd-k8s-master                      1/1       Running   0          1m        192.168.0.39   k8s-master   <none>
kube-apiserver-k8s-master            1/1       Running   0          1m        192.168.0.39   k8s-master   <none>
kube-controller-manager-k8s-master   1/1       Running   0          1m        192.168.0.39   k8s-master   <none>
kube-flannel-ds-amd64-pwnmc          1/1       Running   0          5m        192.168.0.39   k8s-master   <none>
kube-proxy-bh54w                     1/1       Running   0          23h       192.168.0.39   k8s-master   <none>
kube-scheduler-k8s-master            1/1       Running   0          1m        192.168.0.39   k8s-master   <none>
```  

可以看到，已经多了一个`quay.io/coreos/flannel`的 flannel容器和一个`kube-flannel-ds-amd64-pwnmc` 的pod 资源。此时再使用 `kubectl  get pods`来查看集群状态，就会发现,原来的NotReady已经切换成了Ready。  

```shell
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS    ROLES     AGE       VERSION
k8s-master   Ready     master    23h       v1.11.2
```  

### 其他节点加入到集群中

在其他节点中执行下面的加入到集群中的命令，需要手动指定忽略 swap 错误。  

```shell
kubeadm join 192.168.0.39:6443 --token fw3j8a.ynw60celvzihylou --discovery-token-ca-cert-hash sha256:700ce85cde3a8cae39c4c957269090f20062ed5b397d214f261b33ef68404145 --ignore-preflight-errors=Swap 
```  

执行成功之后会输出下面的内容 。

```shell 
..........
This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```  
此时在master节点上执行`kubectl get nodes`命令就可以查看 目前集群的状态。此时nodes 节点上docker 需要去网络上下载相应的镜像文件，所以需要稍等一下。如果前面使用了第一种方式去下载docker镜像的话，这里三个节点也需要手动去下载镜像，需要下载下面这些镜像。  

```shell
 ~]# docker images 
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy-amd64   v1.11.2             46a3cd725628        3 weeks ago         97.8MB
quay.io/coreos/flannel        v0.10.0-amd64       f0fad859c909        7 months ago        44.6MB
k8s.gcr.io/pause              3.1                 da86e6ba6ca1        8 months ago        742kB  
```

全部节点加入完成之后，可以在master节点上查看集群节点

```shell
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS    ROLES     AGE       VERSION
k8s-master   Ready     master    23h       v1.11.2
k8s-node1    Ready     <none>    28m       v1.11.2
k8s-node2    Ready     <none>    27m       v1.11.2
k8s-node3    Ready     <none>    27m       v1.11.2
```  

此时，再查看一下pod的运行状况 

```shell
[root@k8s-master ~]# kubectl get pods -n kube-system
NAME                                 READY     STATUS    RESTARTS   AGE
coredns-78fcdf6894-khlkd             1/1       Running   1          23h
coredns-78fcdf6894-qqrnc             1/1       Running   1          23h
etcd-k8s-master                      1/1       Running   1          42m
kube-apiserver-k8s-master            1/1       Running   1          42m
kube-controller-manager-k8s-master   1/1       Running   1          42m
kube-flannel-ds-amd64-9b4zg          1/1       Running   0          28m
kube-flannel-ds-amd64-jd24m          1/1       Running   3          28m
kube-flannel-ds-amd64-pwnmc          1/1       Running   2          46m
kube-flannel-ds-amd64-zk7xj          1/1       Running   0          29m
kube-proxy-49gz8                     1/1       Running   0          28m
kube-proxy-bh54w                     1/1       Running   1          23h
kube-proxy-m4gcz                     1/1       Running   0          29m
kube-proxy-x6p74                     1/1       Running   0          28m
kube-scheduler-k8s-master            1/1       Running   1          42m  
```

通过分析上面pod的运行状况我们可以发现正在运行的所有pod节点中一共有4个flannel以及4个proxy.这就表明，每个节点上都会安装。而其他的应该都安装了master节点上。   

**至此，基于V1.11 版本的kubernetes集群搭建就完成了。**   














