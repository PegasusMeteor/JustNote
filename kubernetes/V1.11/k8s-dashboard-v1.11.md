## kubernetes dashboard认证及分级授权

kubernetes 提供了Dashboard来管理kubernetes集群，下面我们开安装一下kubenetes Dashboard。github上的官方站点[https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)   



### 安装Dashboard  
在master节点上执行下面的命令。  
```shell
~]# kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
secret/kubernetes-dashboard-certs created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
```  
同样的，命令执行成功之后，并不会立即生效，需要给kubernetes 创建pod的时间。一段时间时候使用`kubectl get pods -n kube-system` 命令查看，会发现，多了一个`kubernetes-dashboard-767dc7d4d-x6x76` 的pod。

```shell  
[root@k8s-master ~]# kubectl get pods -n kube-system
NAME                                   READY     STATUS    RESTARTS   AGE
coredns-78fcdf6894-khlkd               1/1       Running   1          1d
coredns-78fcdf6894-qqrnc               1/1       Running   1          1d
etcd-k8s-master                        1/1       Running   1          1h
kube-apiserver-k8s-master              1/1       Running   1          1h
kube-controller-manager-k8s-master     1/1       Running   1          1h
kube-flannel-ds-amd64-9b4zg            1/1       Running   0          48m
kube-flannel-ds-amd64-jd24m            1/1       Running   4          48m
kube-flannel-ds-amd64-pwnmc            1/1       Running   2          1h
kube-flannel-ds-amd64-zk7xj            1/1       Running   0          49m
kube-proxy-49gz8                       1/1       Running   0          48m
kube-proxy-bh54w                       1/1       Running   1          1d
kube-proxy-m4gcz                       1/1       Running   0          49m
kube-proxy-x6p74                       1/1       Running   0          48m
kube-scheduler-k8s-master              1/1       Running   1          1h
kubernetes-dashboard-767dc7d4d-x6x76   1/1       Running   0          1m
```  
还可以使用下面的命令来查看Dashboard被安装在哪个节点上。  

```shell
[root@k8s-master ~]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                   READY     STATUS    RESTARTS   AGE       IP             NODE         NOMINATED NODE
kube-system   coredns-78fcdf6894-khlkd               1/1       Running   1          1d        10.244.0.5     k8s-master   <none>
kube-system   coredns-78fcdf6894-qqrnc               1/1       Running   1          1d        10.244.0.4     k8s-master   <none>
kube-system   etcd-k8s-master                        1/1       Running   1          1h        192.168.0.39   k8s-master   <none>
kube-system   kube-apiserver-k8s-master              1/1       Running   2          1h        192.168.0.39   k8s-master   <none>
kube-system   kube-controller-manager-k8s-master     1/1       Running   1          1h        192.168.0.39   k8s-master   <none>
kube-system   kube-flannel-ds-amd64-9b4zg            1/1       Running   0          1h        192.168.0.42   k8s-node3    <none>
kube-system   kube-flannel-ds-amd64-jd24m            1/1       Running   4          1h        192.168.0.41   k8s-node2    <none>
kube-system   kube-flannel-ds-amd64-pwnmc            1/1       Running   2          1h        192.168.0.39   k8s-master   <none>
kube-system   kube-flannel-ds-amd64-zk7xj            1/1       Running   0          1h        192.168.0.40   k8s-node1    <none>
kube-system   kube-proxy-49gz8                       1/1       Running   0          1h        192.168.0.42   k8s-node3    <none>
kube-system   kube-proxy-bh54w                       1/1       Running   1          1d        192.168.0.39   k8s-master   <none>
kube-system   kube-proxy-m4gcz                       1/1       Running   0          1h        192.168.0.40   k8s-node1    <none>
kube-system   kube-proxy-x6p74                       1/1       Running   0          1h        192.168.0.41   k8s-node2    <none>
kube-system   kube-scheduler-k8s-master              1/1       Running   1          1h        192.168.0.39   k8s-master   <none>
kube-system   kubernetes-dashboard-767dc7d4d-x6x76   1/1       Running   0          51m       10.244.3.2     k8s-node3    <none>
```
可以看到，Dashboard被安装在了node3节点上。  

接下来在master节点上执行`kubectl proxy`命令就可以在master主机上通过 `http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/` 进行访问了。但是我们的所有节点都是在虚拟机中完成的，不能直接访问，所以接下来我们需要将修改一下配置，以便能够通过浏览器进行远程访问。   

参考地址 [https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above](https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above)


执行 `kubectl -n kube-system edit service kubernetes-dashboard`命令将 `type: ClusterIP` 改成 `type: NodePort`。  

```shell
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":"kube-system"},"spec":{"ports":[{"port":443,"targetPort":8443}],"selector":{"k8s-app":"kubernetes-dashboard"}}}
  creationTimestamp: 2018-08-30T14:47:35Z
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
  resourceVersion: "21084"
  selfLink: /api/v1/namespaces/kube-system/services/kubernetes-dashboard
  uid: a21dd4ea-ac63-11e8-9902-000c296af098
spec:
  clusterIP: 10.107.5.5
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```  


然后执行 `kubectl -n kube-system get service kubernetes-dashboard`就可以查看到端口映射了。  

```shell
[root@k8s-master ~]# kubectl -n kube-system get service kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.107.5.5   <none>        443:32143/TCP   10m
```

这时通过  https://<master-ip>:32143 来进行访问了。`master-ip`，应该就是master节点的ip地址。如果不记得了话可以通过下面的命令来进行查看。  

```shell
[root@k8s-master ~]# kubectl cluster-info
Kubernetes master is running at https://192.168.0.39:6443
KubeDNS is running at https://192.168.0.39:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```    
通过官方网站还介绍，如果我们是在多节点群集上使用NodePort公开Dashboard，则必须找到运行Dashboard的节点的IP以访问它。
前面通过`kubectl get pods --all-namespaces -o wide`,我们已经看到，Dashboard运行在第三个节点上，所以我们应该通过https://<node3-ip>:32143 来进行访问。   

**目前Dashboard不可用，未完待续**
