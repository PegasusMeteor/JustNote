## kubernetes pod 控制器三

### pod 控制器

自主式pod：不是由控制器来管理和控制的，手动控制和管理的
将来在实际使用pod的时候，是不会使用自主式pod的。


下面几个关键词多查看一下kubernetes的官方网站。  
**后期需要将每个控制器单独写一篇文章出来，并辅以案例分析。**

#### ReplicaSet

```shell

[root@k8s-master ~]# kubectl explain rs
KIND:     ReplicaSet
VERSION:  extensions/v1beta1

DESCRIPTION:
     DEPRECATED - This group version of ReplicaSet is deprecated by
     apps/v1beta2/ReplicaSet. See the release notes for more information.
     ReplicaSet ensures that a specified number of pod replicas are running at
     any given time.

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
     If the Labels of a ReplicaSet are empty, they are defaulted to be the same
     as the Pod(s) that the ReplicaSet manages. Standard object's metadata. More
     info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

   spec <Object>
     Spec defines the specification of the desired behavior of the ReplicaSet.
     More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

   status       <Object>
     Status is the most recently observed status of the ReplicaSet. This data
     may be out of date by some window of time. Populated by the system.
     Read-only. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status


```

```shell
[root@k8s-master manifests]# kubectl get rs
NAME                     DESIRED   CURRENT   READY     AGE
myapp                    2         2         0         9s
nginx-deploy-5b595999    3         3         3         10d
nginx-deploy-b6659bbdb   0         0         0         10d
[root@k8s-master manifests]# cat rs-demo.yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp
  namespace: default 
spec:
  replicas: 2
  selector:
    matchLabels:  
      app: myapp
      release: canary
  template: 
    metadata:
      name: myapp-pod
      labels: 
        app: myapp
        release: canary 
        environment: qa
    spec: 
      containers:
        - name: myapp-container
          image: nginx:latest
          ports: 
            - name: http
              containerPort: 80 
```


```shell
[root@k8s-master manifests]# kubectl create -f rs-demo.yaml 
replicaset.apps/myapp created
[root@k8s-master manifests]# kubectl get rs
NAME                     DESIRED   CURRENT   READY     AGE
myapp                    2         2         0         9s
[root@k8s-master manifests]# kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
liveness-httpget-pod          1/1       Running   1          2d
myapp-4s8xd                   1/1       Running   0          1m
myapp-92pkj                   1/1       Running   0          1m
nginx-deploy-5b595999-7rgsh   1/1       Running   0          1h
nginx-deploy-5b595999-7t6q7   1/1       Running   0          1h
nginx-deploy-5b595999-xnwph   1/1       Running   1          10d
poststart-pod                 1/1       Running   2          2d
readiness-httpget-pod         1/1       Running   1          2d

```


```shell
[root@k8s-master manifests]# kubectl describe pod myapp-4s8xd
Name:               myapp-4s8xd
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node2/192.168.0.41
Start Time:         Wed, 31 Oct 2018 23:02:20 +0800
Labels:             app=myapp
                    environment=qa
                    release=canary
Annotations:        <none>
Status:             Running
IP:                 10.244.2.10
Controlled By:      ReplicaSet/myapp
Containers:
  myapp-container:
    Container ID:   docker://e83e5b0c39eebac88c115b8f49f5cbf418dd4d6f767c1755152400c06fa2f4ef
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:b73f527d86e3461fd652f62cf47e7b375196063bbbd503e853af5be16597cb2e
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 31 Oct 2018 23:03:05 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2wpbt (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-2wpbt:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-2wpbt
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                Message
  ----    ------     ----  ----                -------
  Normal  Scheduled  2m    default-scheduler   Successfully assigned default/myapp-4s8xd to k8s-node2
  Normal  Pulling    2m    kubelet, k8s-node2  pulling image "nginx:latest"
  Normal  Pulled     1m    kubelet, k8s-node2  Successfully pulled image "nginx:latest"
  Normal  Created    1m    kubelet, k8s-node2  Created container
  Normal  Started    1m    kubelet, k8s-node2  Started container
```

还可以使用 `kubectl edit rs myapp` 这个命令来实时修改 

#### Deployment 

Deployment 是建构再rs之上的。  


#### DaemonSet 

#### Job

#### Cronjob

#### StatefulSet




