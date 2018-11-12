## kubernets 存储卷

与docker存储卷类似，但是k8s的存储卷是归属于pod的。容器中可以绑定这个目录来使用。

### 存储卷类型

kubenetes 支持的存储类型 

```shell
[root@k8s-master ~]# kubectl explain pod.spec.volumes
KIND:     Pod
VERSION:  v1

RESOURCE: volumes <[]Object>

DESCRIPTION:
     List of volumes that can be mounted by containers belonging to the pod.
     More info: https://kubernetes.io/docs/concepts/storage/volumes

     Volume represents a named volume in a pod that may be accessed by any
     container in the pod.

FIELDS:
   awsElasticBlockStore <Object>
     AWSElasticBlockStore represents an AWS Disk resource that is attached to a
     kubelet's host machine and then exposed to the pod. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#awselasticblockstore

   azureDisk    <Object>
     AzureDisk represents an Azure Data Disk mount on the host and bind mount to
     the pod.

   azureFile    <Object>
     AzureFile represents an Azure File Service mount on the host and bind mount
     to the pod.

   cephfs       <Object>
     CephFS represents a Ceph FS mount on the host that shares a pod's lifetime

   cinder       <Object>
     Cinder represents a cinder volume attached and mounted on kubelets host
     machine More info:
     https://releases.k8s.io/HEAD/examples/mysql-cinder-pd/README.md

   configMap    <Object>
     ConfigMap represents a configMap that should populate this volume

   downwardAPI  <Object>
     DownwardAPI represents downward API about the pod that should populate this
     volume

   emptyDir     <Object>
     EmptyDir represents a temporary directory that shares a pod's lifetime.
     More info: https://kubernetes.io/docs/concepts/storage/volumes#emptydir

   fc   <Object>
     FC represents a Fibre Channel resource that is attached to a kubelet's host
     machine and then exposed to the pod.

   flexVolume   <Object>
     FlexVolume represents a generic volume resource that is
     provisioned/attached using an exec based plugin.

   flocker      <Object>
     Flocker represents a Flocker volume attached to a kubelet's host machine.
     This depends on the Flocker control service being running

   gcePersistentDisk    <Object>
     GCEPersistentDisk represents a GCE Disk resource that is attached to a
     kubelet's host machine and then exposed to the pod. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#gcepersistentdisk

   gitRepo      <Object>
     GitRepo represents a git repository at a particular revision. DEPRECATED:
     GitRepo is deprecated. To provision a container with a git repo, mount an
     EmptyDir into an InitContainer that clones the repo using git, then mount
     the EmptyDir into the Pod's container.

   glusterfs    <Object>
     Glusterfs represents a Glusterfs mount on the host that shares a pod's
     lifetime. More info:
     https://releases.k8s.io/HEAD/examples/volumes/glusterfs/README.md

   hostPath     <Object>
     HostPath represents a pre-existing file or directory on the host machine
     that is directly exposed to the container. This is generally used for
     system agents or other privileged things that are allowed to see the host
     machine. Most containers will NOT need this. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#hostpath

   iscsi        <Object>
     ISCSI represents an ISCSI Disk resource that is attached to a kubelet's
     host machine and then exposed to the pod. More info:
     https://releases.k8s.io/HEAD/examples/volumes/iscsi/README.md

   name <string> -required-
     Volume's name. Must be a DNS_LABEL and unique within the pod. More info:
     https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names

   nfs  <Object>
     NFS represents an NFS mount on the host that shares a pod's lifetime More
     info: https://kubernetes.io/docs/concepts/storage/volumes#nfs

   persistentVolumeClaim        <Object>
     PersistentVolumeClaimVolumeSource represents a reference to a
     PersistentVolumeClaim in the same namespace. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

   photonPersistentDisk <Object>
     PhotonPersistentDisk represents a PhotonController persistent disk attached
     and mounted on kubelets host machine

   portworxVolume       <Object>
     PortworxVolume represents a portworx volume attached and mounted on
     kubelets host machine

   projected    <Object>
     Items for all in one resources secrets, configmaps, and downward API

   quobyte      <Object>
     Quobyte represents a Quobyte mount on the host that shares a pod's lifetime

   rbd  <Object>
     RBD represents a Rados Block Device mount on the host that shares a pod's
     lifetime. More info:
     https://releases.k8s.io/HEAD/examples/volumes/rbd/README.md

   scaleIO      <Object>
     ScaleIO represents a ScaleIO persistent volume attached and mounted on
     Kubernetes nodes.

   secret       <Object>
     Secret represents a secret that should populate this volume. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#secret

   storageos    <Object>
     StorageOS represents a StorageOS volume attached and mounted on Kubernetes
     nodes.

   vsphereVolume        <Object>
     VsphereVolume represents a vSphere volume attached and mounted on kubelets
     host machine

[root@k8s-master ~]# 

```

### 创建一个emptyDir存储卷

在pod中创建一个存储卷，并在容器中挂载存储卷。一个存储卷，可以被多个容器来挂载和使用，下面以emptyDir为例。  

```shell
[root@k8s-master ~]# cat  manifests/volumes/pod-volume-demo.yaml             
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-demo 
  namespace: default
  labels:
    app: myapp
    tier: frontend 
spec:
  containers:
#  可以指定多个image,自己的image需要按照下面的格式
#  - name: myapp
#    image: ikubernetes/myapp:v1
  - name: nginx
    image: nginx:1.13
    ports: 
    - name: http
      containerPort: 80
# 挂载pod上哪个卷
    volumeMounts:
    - name: html
      mountPath: /data/web/html
  - name: busybox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: html 
      mountPath: /data/
    command: 
    - "/bin/sh"
    - "-c"
    - "sleep 7200" 
  volumes:
  - name: html 
    emptyDir: {}


[root@k8s-master ~]# kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
myapp-deploy-6d96b6cc45-4tk7h   1/1       Running   1          3d
myapp-deploy-6d96b6cc45-lz9f4   1/1       Running   1          3d
myapp-deploy-6d96b6cc45-mh2nk   1/1       Running   1          3d
pod-volume-demo                 2/2       Running   0          59s

```

接下来，我们连入pod中的busybox容器，并在挂载的存储卷内部写点内容。  


```shell
[root@k8s-master ~]# kubectl exec -it pod-volume-demo  -c busybox -- /bin/sh
/ # date
Mon Nov 12 14:26:28 UTC 2018
/ # echo hello >> /data/index.html 
/ # 

```
我们连入pod中的nginx容器，并在挂载的存储卷内部写点内容。  

```shell
[root@k8s-master ~]# kubectl exec -it pod-volume-demo  -c nginx -- /bin/sh       
# cd /data/web/html
# echo helloworld > index.html
```

此时，我们再切换回 busybox容器，就会发现，两个容器共享了一个存储空间。两个index.html文件中的内容是一样的。


### gitRepo 存储卷

gitRepo 本质上还是emptyDir来实现的。会将远程git上的代码，拉取到pod存储卷中。创建gitRepo的前提依据是宿主机安装了git。  


### hostPath 存储卷

hostPath 存储卷，在pod被删除之后，宿主机上的数据是不会丢失的。如果，后期创建的pod，使用了这个存储卷，就可以直接使用里面的数据。 

首先编写一个清单文件。  

```shell
[root@k8s-master volumes]# cat pod-volume-hostpath.yaml 
apiVersion: v1
kind: Pod
metadata: 
  name: pod-vol-hostpath
  namespace: default 
spec:
  containers: 
  - name: myapp
    image: nginx:1.13
    volumeMounts:
    - name: html 
      mountPath: /usr/share/nginx/html/
  volumes: 
  - name: html 
    hostPath: 
      path: /data/pod/volume1
      type: DirectoryOrCreate 
```

接下来，我们在每个节点上先事先创建一下，我们需要挂载的目录 `/data/pod/volume1`,并在里面写入不同的内容。因为我们不能够确定被创建的pod会被调度到哪个节点上。     

然后，我们创建一下pod，并访问一下pod上的资源。

```shell

[root@k8s-master volumes]# kubectl apply -f pod-volume-hostpath.yaml
pod/pod-vol-hostpath created
[root@k8s-master volumes]# kubectl get pods -o wide
NAME               READY     STATUS    RESTARTS   AGE       IP            NODE        NOMINATED NODE
pod-vol-hostpath   1/1       Running   0          9s        10.244.3.27   k8s-node3   <none>
```
从上面可以看出，这个pod创建在node3节点上。下面我们来访问一下试试。   

```shell
[root@k8s-master volumes]# curl 10.244.3.27
node3
```

然后我们可以将这个pod删除之后，再进行访问，就可以发现，被创建在不同节点上的pod，由于指定的hostPath不同，所以访问到的数据也有所不同。   

**但是，这样的话，在进行跨节点调度时，不同节点之间的数据不能同步，这将导致很不方便。**

### nfs 存储卷

解决不同节点之间访问数据不一致的问题，就可以采用网络存储。例如NFS.  

**首先准备一个网络存储**
由于笔者硬件资源有限，所以我们选定其中一个节点来作为nfs存储。但是其他的节点想要使用nfs存储的话，需要能够挂载网络存储。因此在实际使用中，除了nfs存储节点，其余的k8s集群中凡是想要使用nfs存储的节点都要nfs-utils.   

在所有的节点中安装 nfs-utils.
```shell
~]#  yum install nfs-utils -y
```
选定node1为nfs存储节点,创建存储目录。并将此目录共享给192.168.0.0/16 网络中主机使用。配置完成后启动nfs。   

```shell
[root@k8s-node1 ~]# mkdir /data/volumes -pv
[root@k8s-node1 ~]# cat /etc/exports
/data/volumes 192.168.0.0/16(rw,no_root_squash)
[root@k8s-node1 ~]# systemctl start nfs

```

可以在其他节点，尝试先手动挂载一个nfs目录试试。可以发现是没有问题的。   

```shell
[root@k8s-node2 ~]# mount -t nfs 192.168.0.40:/data/volumes /mnt   
[root@k8s-node2 ~]# umount /mnt
```

**创建一个资源清单文件** 

首先创建一个清单文件，这个文件与之前的hostPath文件很相像。  

```shell
[root@k8s-master volumes]# cat pod-volume-nfs.yaml 
apiVersion: v1
kind: Pod
metadata: 
  name: pod-vol-nfs
  namespace: default 
spec:
  containers: 
  - name: myapp
    image: nginx:1.13
    volumeMounts:
    - name: html 
      mountPath: /usr/share/nginx/html/
  volumes: 
  - name: html 
    nfs: 
      path: /data/volumes
# 这里直接写的IP地址，也可以直接写主机名
      server: 192.168.0.40

```
下面创建一下这个pod。  

```shell
[root@k8s-master volumes]# kubectl apply -f pod-volume-nfs.yaml 
pod/pod-vol-nfs created
[root@k8s-master volumes]# kubectl get pods -o wide 
NAME          READY     STATUS    RESTARTS   AGE       IP            NODE        NOMINATED NODE
pod-vol-nfs   1/1       Running   0          7s        10.244.2.24   k8s-node2   <none>
```
上面可以看出，这个pod被创建在 node2节点上。但是，我们在之前创建的nfs目录中创建几个文件，然后我们访问一下node2上这个容器来看一下。  

```shell
[root@k8s-node1 ~]# echo nfs-server >> /data/volumes/index.html 
[root@k8s-node1 ~]# curl 10.244.2.24
nfs-server
```
**node1在这里担任两个角色，分别是nfs-server，以及k8s集群中的node1**

同样，这时，我们将刚刚创建的pod删除掉，再重新创建，就会发现虽然pod运行的节点变了，但是访问内容应该没变。或者即便pod被删除了，数据依然存在。  

**但是nfs宕机了，怎么办呢？答案就是使用分布式存储。**  


### PVC 

什么是PVC？ 可以先看一下解释。  

```shell

[root@k8s-master ~]# kubectl explain pvc
KIND:     PersistentVolumeClaim
VERSION:  v1

DESCRIPTION:
     PersistentVolumeClaim is a user's request for and claim to a persistent
     volume

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

   spec <Object>
     Spec defines the desired characteristics of a volume requested by a pod
     author. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

   status       <Object>
     Status represents the current information/status of a persistent volume
     claim. Read-only. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

```  














