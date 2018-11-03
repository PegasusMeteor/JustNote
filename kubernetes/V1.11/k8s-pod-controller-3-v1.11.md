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

```shell
[root@k8s-master ~]# kubectl explain deploy 
KIND:     Deployment
VERSION:  extensions/v1beta1

DESCRIPTION:
     DEPRECATED - This group version of Deployment is deprecated by
     apps/v1beta2/Deployment. See the release notes for more information.
     Deployment enables declarative updates for Pods and ReplicaSets.

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
     Standard object metadata.

   spec <Object>
     Specification of the desired behavior of the Deployment.

   status       <Object>
     Most recently observed status of the Deployment.

```
下面我们来定义一个deployment 来看看实际的效果。


```shell
[root@k8s-master manifests]# cat deploy-demo.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
        image: nginx:1.13.8
        ports:
        - name: http
          containerPort: 80 
        
[root@k8s-master manifests]# kubectl apply -f deploy-demo.yaml 
deployment.apps/myapp-deploy created
[root@k8s-master manifests]# kubectl get deploy         
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
myapp-deploy   2         2         2            0           1m

```

可以试用patch 命令来给我们的deployment 来动态的打补丁。但是这种方式并不会真正的修改我们自定义的配置文件。

```shell
[root@k8s-master manifests]# kubectl patch deployment myapp-deploy -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'
deployment.extensions/myapp-deploy patched
```
还可以直接对镜像进行修改
```shell
[root@k8s-master manifests]# kubectl set image deployment myapp-deploy myapp=nginx:latest 
deployment.extensions/myapp-deploy image updated
```



#### DaemonSet 
在整个集群的每一个节点上，只运行一个指定的pod副本。

```shell
[root@k8s-master manifests]# kubectl explain ds
KIND:     DaemonSet
VERSION:  extensions/v1beta1

DESCRIPTION:
     DEPRECATED - This group version of DaemonSet is deprecated by
     apps/v1beta2/DaemonSet. See the release notes for more information.
     DaemonSet represents the configuration of a daemon set.

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
     The desired behavior of this daemon set. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

   status       <Object>
     The current status of this daemon set. This data may be out of date by some
     window of time. Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status
```


```shell
[root@k8s-master manifests]# cat ds-demo.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat-ds
  namespace: default
spec:
  selector:
    matchLabels:
      app: filebeat
      release: stable
  template:
    metadata:
      labels:
        app: filebeat
        release: stable
    spec:
      containers:
      - name: filebeat
        image: dev_ops/filebeat:latest
        env: 
        - name: REDIS_HOST
          value: redis.default.cluster.svc.cluster.local 
        - name: REDIS_LOG_LEVEL
          value: info 
         
[root@k8s-master manifests]# kubectl apply -f ds-demo.yaml 
daemonset.apps/myapp-ds created


```

#### Job

#### Cronjob

#### StatefulSet




