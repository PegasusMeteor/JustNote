
## 使用kubeadm创建高可用集群



本章介绍了使用kubeadm通过两种不同的方法来建立一个高可用的Kubernetes集群:

- 压缩master节点(master与etcd安装在一起). 这个方法需要更小的资源. etcd 的成员与 控制台节点共存.
- 扩展的etcd集群. 这个方法需要更多的基础资源. 控制台节点与etcd成员是分开的.

Kubernetes 集群需要 1.11 或者 更新的版本. 并且需要明白的是，使用kubeadm创建高可用集群仍处在实验阶段。例如，升级集群，就可能会遇到不少问题. 此时可以尝试使用另外一种方式，或者进行反馈.


> **注意**: 本章并没有介绍在云提供商上运行kubernetes集群.在云环境中，这两种方式均不能与负载均衡器和动态挂在卷很好的协作。

## 本章目录
- 开始之前
- 第一个步骤(两个方法都要)
- Stacked control plane nodes
- 扩展 etcd
- 引导控制台之后的一些通用工作


## 开始之前

这两种方法都需要下面这些基础设施:

- 3个master节点 [kubeadm最小要求]() 
- 3台worker节点 [kubeadm最小要求]() 
- 集群中所有设备网络联通 (公共或者私有网络均可)
- 所有节点之间SSH互通 
- 所有设备上的sudo权限

如果是扩展的ETCD集群，还需要下面:

- 额外的3台设备给etcd成员


> **Note**: 下面的例子使用 Calico 作为 Pod networking provider. 如果使用了另外一种 network provider，确保已经按需调整了默认值。


## 第一个步骤(for both methods) 


> **Note**: 下面的命令都应该以root用户来运行，不管是控制台节点还是etcd节点。


- 找到 pod CIDR. 可以查看 [安装 POD 网络组件](create-cluster-kubeadm.html).
   本次示例使用Calico网络 , 所以 pod CIDR 是 `192.168.0.0/16`.

### 配置 SSH

1.  在主要设备上启用ssh客户端，要求可以访问到系统中其他所有节点:

     ```
     eval $(ssh-agent)
     ```

1.  调价SSH 标识到配置文件:

     ```
     ssh-add ~/.ssh/path_to_private_key
     ```

1.  检查各个节点之间的SSH链接.

    - ssh 链接时，确保加上了 `-A` 参数:

      ```
      ssh -A 10.0.0.7
      ```

    - 任何一个节点上使用sudo时，确保已经修改了环境，让SSH 转发能够起作用:

      ```
      sudo -E -s
      ```

### 为kube-apiserver创建一个负载均衡器


> **Note**: 负载均衡器有很多种配置，下面的例子只是其中一种. 你的集群可能需要不同的配置.


1.  创建一个名字能够被DNS解析的kube-apiserver负载均衡器.

    - 在云环境中，需要把控制台节点放在TCP转发负载均衡器之后。 这个负载均衡器会将请求分发给它列表中健康运行的控制台节点。apiserver 的健康性检查针对 kube-apiserver 端口（默认`:6443`）基于TCP协议的。

    - 在云环境中，不建议直接使用ip地址.

    - 负载均衡器必须能够通过apiserver监听的端口与所有的控制台节点进行通信。 而且还必须允许访问这个监听端口的流量通过.

1.  将第一个控制台节点加入负载均衡器，并测试联通性:

    ```sh
    nc -v LOAD_BALANCER_IP PORT
    ```

    因为apiserver还未运行，此时会出现链接请求被拒绝的错误。如果是超时错误的话,意味着负载均衡器还不能与控制台节点进行通信。 如果发生超时的话,就需要重新配置负载均衡器让它能够与控制台节点正常通信.

1.  将剩余的控制台节点加入到负载均衡器中.

## Stacked control plane nodes

### 引导第一个 stacked control plane node

1.  创建一个 `kubeadm-config.yaml` 模板文件:

        apiVersion: kubeadm.k8s.io/v1alpha2
        kind: MasterConfiguration
        kubernetesVersion: v1.11.0
        apiServerCertSANs:
        - "LOAD_BALANCER_DNS"
        api:
            controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
        etcd:
          local:
            extraArgs:
              listen-client-urls: "https://127.0.0.1:2379,https://CP0_IP:2379"
              advertise-client-urls: "https://CP0_IP:2379"
              listen-peer-urls: "https://CP0_IP:2380"
              initial-advertise-peer-urls: "https://CP0_IP:2380"
              initial-cluster: "CP0_HOSTNAME=https://CP0_IP:2380"
            serverCertSANs:
              - CP0_HOSTNAME
              - CP0_IP
            peerCertSANs:
              - CP0_HOSTNAME
              - CP0_IP
        networking:
            # This CIDR is a Calico default. Substitute or remove for your CNI provider.
            podSubnet: "192.168.0.0/16"


1.  用适合集群的值替换模板中的下列变量值:

    * `LOAD_BALANCER_DNS`
    * `LOAD_BALANCER_PORT`
    * `CP0_HOSTNAME`
    * `CP0_IP`

1.  运行 `kubeadm init --config kubeadm-config.yaml`

### 拷贝需要的文件到其他的 control plane nodes

当运行 `kubeadm init` 时会产生下面这些证书以及其他需要的文件.
将这些文件copy到其他的控制台节点:

- `/etc/kubernetes/pki/ca.crt`
- `/etc/kubernetes/pki/ca.key`
- `/etc/kubernetes/pki/sa.key`
- `/etc/kubernetes/pki/sa.pub`
- `/etc/kubernetes/pki/front-proxy-ca.crt`
- `/etc/kubernetes/pki/front-proxy-ca.key`
- `/etc/kubernetes/pki/etcd/ca.crt`
- `/etc/kubernetes/pki/etcd/ca.key`

拷贝 admin kubeconfig 到其他的控制台节点:

- `/etc/kubernetes/admin.conf`

在此例中，用其他的控制台节点的ip地址替换掉`CONTROL_PLANE_IPS` 的值.

```sh
USER=ubuntu # customizable
CONTROL_PLANE_IPS="10.0.0.7 10.0.0.8"
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
    scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
    scp /etc/kubernetes/admin.conf "${USER}"@$host:
done
```


> **注意**: 你的陪着可能和这个例子有所不同.


### 添加第二个 stacked control plane node

1.  创建第二个不同的 `kubeadm-config.yaml` 模板文件:

        apiVersion: kubeadm.k8s.io/v1alpha2
        kind: MasterConfiguration
        kubernetesVersion: v1.11.0
        apiServerCertSANs:
        - "LOAD_BALANCER_DNS"
        api:
            controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
        etcd:
          local:
            extraArgs:
              listen-client-urls: "https://127.0.0.1:2379,https://CP1_IP:2379"
              advertise-client-urls: "https://CP1_IP:2379"
              listen-peer-urls: "https://CP1_IP:2380"
              initial-advertise-peer-urls: "https://CP1_IP:2380"
              initial-cluster: "CP0_HOSTNAME=https://CP0_IP:2380,CP1_HOSTNAME=https://CP1_IP:2380"
              initial-cluster-state: existing
            serverCertSANs:
              - CP1_HOSTNAME
              - CP1_IP
            peerCertSANs:
              - CP1_HOSTNAME
              - CP1_IP
        networking:
            # This CIDR is a calico default. Substitute or remove for your CNI provider.
            podSubnet: "192.168.0.0/16"

1.  用适合集群的值替换模板中的下列变量值:

    - `LOAD_BALANCER_DNS`
    - `LOAD_BALANCER_PORT`
    - `CP0_HOSTNAME`
    - `CP0_IP`
    - `CP1_HOSTNAME`
    - `CP1_IP`

1.  将复制过来的文件放置到合适和目录下:

      ```sh
      USER=ubuntu # customizable
      mkdir -p /etc/kubernetes/pki/etcd
      mv /home/${USER}/ca.crt /etc/kubernetes/pki/
      mv /home/${USER}/ca.key /etc/kubernetes/pki/
      mv /home/${USER}/sa.pub /etc/kubernetes/pki/
      mv /home/${USER}/sa.key /etc/kubernetes/pki/
      mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
      mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
      mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
      mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
      mv /home/${USER}/admin.conf /etc/kubernetes/admin.conf
      ```

1.  运行kubeadm 命令来引导 kubelet:

      ```sh
      kubeadm alpha phase certs all --config kubeadm-config.yaml
      kubeadm alpha phase kubelet config write-to-disk --config kubeadm-config.yaml
      kubeadm alpha phase kubelet write-env-file --config kubeadm-config.yaml
      kubeadm alpha phase kubeconfig kubelet --config kubeadm-config.yaml
      systemctl start kubelet
      ```

1.  运行下面的命令将节点加入到 etcd 集群:

      ```sh
      export CP0_IP=10.0.0.7
      export CP0_HOSTNAME=cp0
      export CP1_IP=10.0.0.8
      export CP1_HOSTNAME=cp1

      export KUBECONFIG=/etc/kubernetes/admin.conf 
      kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP1_HOSTNAME} https://${CP1_IP}:2380
      kubeadm alpha phase etcd local --config kubeadm-config.yaml
      ```

    在一个节点加入到了正在运行的etcd集群之后，以及下一个新的节点加入到这个etcd集群之前，这个命令将会导致ETCD集群有一小段时间不可用。.

1.  部署控制台组件，并将这个节点标识为master节点。:

      ```sh
      kubeadm alpha phase kubeconfig all --config kubeadm-config.yaml
      kubeadm alpha phase controlplane all --config kubeadm-config.yaml
      kubeadm alpha phase mark-master --config kubeadm-config.yaml
      ```

### 添加第三个 stacked control plane node

1.  创建第三个不同的 `kubeadm-config.yaml` 模板文件:

        apiVersion: kubeadm.k8s.io/v1alpha2
        kind: MasterConfiguration
        kubernetesVersion: v1.11.0
        apiServerCertSANs:
        - "LOAD_BALANCER_DNS"
        api:
            controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
        etcd:
          local:
            extraArgs:
              listen-client-urls: "https://127.0.0.1:2379,https://CP2_IP:2379"
              advertise-client-urls: "https://CP2_IP:2379"
              listen-peer-urls: "https://CP2_IP:2380"
              initial-advertise-peer-urls: "https://CP2_IP:2380"
              initial-cluster: "CP0_HOSTNAME=https://CP0_IP:2380,CP1_HOSTNAME=https://CP1_IP:2380,CP2_HOSTNAME=https://CP2_IP:2380"
              initial-cluster-state: existing
            serverCertSANs:
              - CP2_HOSTNAME
              - CP2_IP
            peerCertSANs:
              - CP2_HOSTNAME
              - CP2_IP
        networking:
            # This CIDR is a calico default. Substitute or remove for your CNI provider.
            podSubnet: "192.168.0.0/16"

1. 用适合集群的值替换模板中的下列变量值:

    - `LOAD_BALANCER_DNS`
    - `LOAD_BALANCER_PORT`
    - `CP0_HOSTNAME`
    - `CP0_IP`
    - `CP1_HOSTNAME`
    - `CP1_IP`
    - `CP2_HOSTNAME`
    - `CP2_IP`

1.  Move the copied files to the correct locations:

      ```sh
      USER=ubuntu # customizable
      mkdir -p /etc/kubernetes/pki/etcd
      mv /home/${USER}/ca.crt /etc/kubernetes/pki/
      mv /home/${USER}/ca.key /etc/kubernetes/pki/
      mv /home/${USER}/sa.pub /etc/kubernetes/pki/
      mv /home/${USER}/sa.key /etc/kubernetes/pki/
      mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
      mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
      mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
      mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
      mv /home/${USER}/admin.conf /etc/kubernetes/admin.conf
      ```

1.  Run the kubeadm phase commands to bootstrap the kubelet:

      ```sh
      kubeadm alpha phase certs all --config kubeadm-config.yaml
      kubeadm alpha phase kubelet config write-to-disk --config kubeadm-config.yaml
      kubeadm alpha phase kubelet write-env-file --config kubeadm-config.yaml
      kubeadm alpha phase kubeconfig kubelet --config kubeadm-config.yaml
      systemctl start kubelet
      ```

1.  运行下面的命令将节点加入到etcd集群中去:

      ```sh
      export CP0_IP=10.0.0.7
      export CP0_HOSTNAME=cp0
      export CP2_IP=10.0.0.9
      export CP2_HOSTNAME=cp2

      export KUBECONFIG=/etc/kubernetes/admin.conf 
      kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP2_HOSTNAME} https://${CP2_IP}:2380
      kubeadm alpha phase etcd local --config kubeadm-config.yaml
      ```

1.  部署控制台组件，并将这个节点标记为master节点:

      ```sh
      kubeadm alpha phase kubeconfig all --config kubeadm-config.yaml
      kubeadm alpha phase controlplane all --config kubeadm-config.yaml
      kubeadm alpha phase mark-master --config kubeadm-config.yaml
      ```

## 扩展的 etcd 集群

### 搭建ETCD集群

- 点击 [用kubeadm建立一个高度可用的etcd集群](setup-ha-etcd-with-kubeadm.html)
   创建一个etcd集群.

### 拷贝需要的文件到其他的控制台节点

创建etcd集群的时候，会产生下面一系列的证书文件. 将他们拷贝到其他的控制台节点:

- `/etc/kubernetes/pki/etcd/ca.crt`
- `/etc/kubernetes/pki/apiserver-etcd-client.crt`
- `/etc/kubernetes/pki/apiserver-etcd-client.key`

在下面的例子中，用我们自己环境中的值替换掉 `USER` 和 `CONTROL_PLANE_HOSTS`的值。

```sh
USER=ubuntu
CONTROL_PLANE_HOSTS="10.0.0.7 10.0.0.8 10.0.0.9"
for host in $CONTROL_PLANE_HOSTS; do
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/apiserver-etcd-client.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/apiserver-etcd-client.key "${USER}"@$host:
done
```

### 建立第一个控制台节点

1.  创建一个 `kubeadm-config.yaml` 模板:

        apiVersion: kubeadm.k8s.io/v1alpha2
        kind: MasterConfiguration
        kubernetesVersion: v1.11.0
        apiServerCertSANs:
        - "LOAD_BALANCER_DNS"
        api:
            controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
        etcd:
            external:
                endpoints:
                - https://ETCD_0_IP:2379
                - https://ETCD_1_IP:2379
                - https://ETCD_2_IP:2379
                caFile: /etc/kubernetes/pki/etcd/ca.crt
                certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
                keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
        networking:
            # This CIDR is a calico default. Substitute or remove for your CNI provider.
            podSubnet: "192.168.0.0/16"

1.  用适合集群的值替换模板中的下列变量值:

    - `LOAD_BALANCER_DNS`
    - `LOAD_BALANCER_PORT`
    - `ETCD_0_IP`
    - `ETCD_1_IP`
    - `ETCD_2_IP`

1.  运行 `kubeadm init --config kubeadm-config.yaml`

### 拷贝需要的文件到合适的路径下

在运行`kubeadm init`命令时会生成下面一系列的证书以及其他需要的文件 .
将这些文件拷贝到其他的控制台节点:

- `/etc/kubernetes/pki/ca.crt`
- `/etc/kubernetes/pki/ca.key`
- `/etc/kubernetes/pki/sa.key`
- `/etc/kubernetes/pki/sa.pub`
- `/etc/kubernetes/pki/front-proxy-ca.crt`
- `/etc/kubernetes/pki/front-proxy-ca.key`

在这个例子中, 用其他控制台节点的ip地址替换掉`CONTROL_PLANE_IPS` 的值.

```sh
USER=ubuntu # customizable
CONTROL_PLANE_IPS="10.0.0.7 10.0.0.8"
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
done
```


> **Note**: 我们自己实验中的配置可能与本例有所不同.


### 建立其他的控制台节点

1.  确认一下拷贝文件的路径.
    你的 `/etc/kubernetes` 应该像下面这个样子:

    - `/etc/kubernetes/pki/apiserver-etcd-client.crt`
    - `/etc/kubernetes/pki/apiserver-etcd-client.key`
    - `/etc/kubernetes/pki/ca.crt`
    - `/etc/kubernetes/pki/ca.key`
    - `/etc/kubernetes/pki/front-proxy-ca.crt`
    - `/etc/kubernetes/pki/front-proxy-ca.key`
    - `/etc/kubernetes/pki/sa.key`
    - `/etc/kubernetes/pki/sa.pub`
    - `/etc/kubernetes/pki/etcd/ca.crt`

1.  在每个控制台节点上运行 `kubeadm init --config kubeadm-config.yaml` ,
    `kubeadm-config.yaml` 是已经提前创建好的.

## 引导了控制台节点之后的通用配置

### 安装pod 网络

参考[安装pod 网络](create-cluster-kubeadm.html) . 确保这符合我们在master 配置文件中指定的pod CIDR 要求。

### 安装 workers


现在，任何一个worker节点都可以通过 `kubeadm init`  返回的命令加入到集群中
