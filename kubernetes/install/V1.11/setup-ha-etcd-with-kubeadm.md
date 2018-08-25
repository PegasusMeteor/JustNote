## 用kubeadm建立一个高可用的etcd集群



Kubeadm 默认在控制台节点上的kubelet管理的静态pod中运行一个单节点的etcd集群. 这不是一个高可用设置，因为etcd集群只包含一个节点而不能包含其他成员导致变得不可用. 创建一个由三个节点组成的高可用etcd集群也可以在使用kubeadm创建kubernetes集群时作为扩展的etcd集群来使用。



## 本章目录

- 开始之前
- 建立集群
- 下一步

## 

* 三台能够通过 2379 和 2380端口互相通讯的主机. 这里使用默认端口. 可以通过 kubeadm 配置文件进行修改.
* 每个主机必须已经安装了 [ docker, kubelet, 和 kubeadm](install-kubeadm.html).
* 一些能够在主机之间拷贝文件的工具. 例如 `ssh` and `scp`.


## 建立集群

通用的做法是在一个节点上生成所有的证书文件，然后将 *必须* 的文件拷贝到其他的节点上。


> **Note:**  kubeadm 包含了所有的必须的加密机制去生成下面描述的证书文件; 这个例子中，不需要其他的额外的加密工具.



1. 将kubelet配置为etcd的服务管理器.

    运行etcd比运行kubernetes简单， 因此你必须重写由kubeadm提供的kubelet单元文件，并创建一个更高优先级的。

    ```sh
    cat << EOF > /etc/systemd/system/kubelet.service.d/20-etcd-service-manager.conf
    [Service]
    ExecStart=
    ExecStart=/usr/bin/kubelet --address=127.0.0.1 --pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true
    Restart=always
    EOF

    systemctl daemon-reload
    systemctl restart kubelet
    ```

1. 为 kubeadm 创建配置文件.

    用下面的脚本，为每台主机创建一个kubeadm配置文件。每台主机上会有一个etcd成员.

    ```sh
    # Update HOST0, HOST1, and HOST2 with the IPs or resolvable names of your hosts
    export HOST0=10.0.0.6
    export HOST1=10.0.0.7
    export HOST2=10.0.0.8

    # Create temp directories to store files that will end up on other hosts.
    mkdir -p /tmp/${HOST0}/ /tmp/${HOST1}/ /tmp/${HOST2}/

    ETCDHOSTS=(${HOST0} ${HOST1} ${HOST2})
    NAMES=("infra0" "infra1" "infra2")

    for i in "${!ETCDHOSTS[@]}"; do
    HOST=${ETCDHOSTS[$i]}
    NAME=${NAMES[$i]}
    cat << EOF > /tmp/${HOST}/kubeadmcfg.yaml
    apiVersion: "kubeadm.k8s.io/v1alpha2"
    kind: MasterConfiguration
    etcd:
        local:
            serverCertSANs:
            - "${HOST}"
            peerCertSANs:
            - "${HOST}"
            extraArgs:
                initial-cluster: infra0=https://${ETCDHOSTS[0]}:2380,infra1=https://${ETCDHOSTS[1]}:2380,infra2=https://${ETCDHOSTS[2]}:2380
                initial-cluster-state: new
                name: ${NAME}
                listen-peer-urls: https://${HOST}:2380
                listen-client-urls: https://${HOST}:2379
                advertise-client-urls: https://${HOST}:2379
                initial-advertise-peer-urls: https://${HOST}:2380
    EOF
    done
    ```

1. 创建证书颁发机构(CA)

    如果已经有了CA，那仅仅需要将 CA的`crt` 和 `key` 文件拷贝到 `/etc/kubernetes/pki/etcd/ca.crt` 和
    `/etc/kubernetes/pki/etcd/ca.key`. 拷贝完成后，跳到下一步.

    如果还没有 CA ，就在 `$HOST0` (生成kubeadm配置文件的主机)运行下面的命令.

    ```
    kubeadm alpha phase certs etcd-ca
    ```

    会生成下面两个文件

    - `/etc/kubernetes/pki/etcd/ca.crt`
    - `/etc/kubernetes/pki/etcd/ca.key`

1. 为每个成员生成证书

    ```sh
    kubeadm alpha phase certs etcd-server --config=/tmp/${HOST2}/kubeadmcfg.yaml
    kubeadm alpha phase certs etcd-peer --config=/tmp/${HOST2}/kubeadmcfg.yaml
    kubeadm alpha phase certs etcd-healthcheck-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
    kubeadm alpha phase certs apiserver-etcd-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
    cp -R /etc/kubernetes/pki /tmp/${HOST2}/
    # cleanup non-reusable certificates
    find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

    kubeadm alpha phase certs etcd-server --config=/tmp/${HOST1}/kubeadmcfg.yaml
    kubeadm alpha phase certs etcd-peer --config=/tmp/${HOST1}/kubeadmcfg.yaml
    kubeadm alpha phase certs etcd-healthcheck-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
    kubeadm alpha phase certs apiserver-etcd-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
    cp -R /etc/kubernetes/pki /tmp/${HOST1}/
    find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

    kubeadm alpha phase certs etcd-server --config=/tmp/${HOST0}/kubeadmcfg.yaml
    kubeadm alpha phase certs etcd-peer --config=/tmp/${HOST0}/kubeadmcfg.yaml
    kubeadm alpha phase certs etcd-healthcheck-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
    kubeadm alpha phase certs apiserver-etcd-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
    # No need to move the certs because they are for HOST0

    # clean up certs that should not be copied off this host
    find /tmp/${HOST2} -name ca.key -type f -delete
    find /tmp/${HOST1} -name ca.key -type f -delete
    ```

1. 拷贝证书和kubeadm配置

    证书已经生成，现在需要把他们移动到各自的主机.

     ```sh
     USER=ubuntu
     HOST=${HOST1}
     scp -r /tmp/${HOST}/* ${USER}@${HOST}:
     ssh ${USER}@${HOST}
     USER@HOST $ sudo -Es
     root@HOST $ chown -R root:root pki
     root@HOST $ mv pki /etc/kubernetes/
     ```

1. 确保所有需要的文件都存在

    在 `$HOST0` 主机上需要的完整文件列表如下:

    ```
    /tmp/${HOST0}
    └── kubeadmcfg.yaml
    ---
    /etc/kubernetes/pki
    ├── apiserver-etcd-client.crt
    ├── apiserver-etcd-client.key
    └── etcd
        ├── ca.crt
        ├── ca.key
        ├── healthcheck-client.crt
        ├── healthcheck-client.key
        ├── peer.crt
        ├── peer.key
        ├── server.crt
        └── server.key
    ```

    在 `$HOST1`:

    ```
    $HOME
    └── kubeadmcfg.yaml
    ---
    /etc/kubernetes/pki
    ├── apiserver-etcd-client.crt
    ├── apiserver-etcd-client.key
    └── etcd
        ├── ca.crt
        ├── healthcheck-client.crt
        ├── healthcheck-client.key
        ├── peer.crt
        ├── peer.key
        ├── server.crt
        └── server.key
    ```

    在 `$HOST2`

    ```
    $HOME
    └── kubeadmcfg.yaml
    ---
    /etc/kubernetes/pki
    ├── apiserver-etcd-client.crt
    ├── apiserver-etcd-client.key
    └── etcd
        ├── ca.crt
        ├── healthcheck-client.crt
        ├── healthcheck-client.key
        ├── peer.crt
        ├── peer.key
        ├── server.crt
        └── server.key
    ```

1. 创建静态pod的清单文件

    既然证书和配置已经就位了，就是时候生成清单文件了。在每台主机上运行`kubeadm`命令为etcd生成清单文件.

    ```sh
    root@HOST0 $ kubeadm alpha phase etcd local --config=/tmp/${HOST0}/kubeadmcfg.yaml
    root@HOST1 $ kubeadm alpha phase etcd local --config=/home/ubuntu/kubeadmcfg.yaml
    root@HOST2 $ kubeadm alpha phase etcd local --config=/home/ubuntu/kubeadmcfg.yaml
    ```

1. 可选的: 检查一下集群的健康状态

    ```sh
    docker run --rm -it \
    --net host \
    -v /etc/kubernetes:/etc/kubernetes quay.io/coreos/etcd:v3.2.18 etcdctl \
    --cert-file /etc/kubernetes/pki/etcd/peer.crt \
    --key-file /etc/kubernetes/pki/etcd/peer.key \
    --ca-file /etc/kubernetes/pki/etcd/ca.crt \
    --endpoints https://${HOST0}:2379 cluster-health
    ...
    cluster is healthy
    ```


我们创建了三个成员的etcd集群之后，就可以继续使用[扩展的etcd](high-availability.html)搭建高可用控制台了。


