## kubernetes V1.11 应用快速入门


### 常用资源介绍
这里简单列举一下，在K8S中会用到的常用资源，这里先不详细展开，后期在用到的时候再进行详细的介绍。  


资源：对象
- workload(工作负载型资源):Pod,ReplicaSet,Deployment,StatefulSet,DaemonSet,Job,CronJob,...
- 服务发现及均衡:Service，Ingress,...
- 配置与存储:Volume,CSI
    - ConfigMap,Secret
    - SownwardAPI
- 集群级资源
    - Namespace,Node,Role,ClusterRole.RoleBinding,ClusterRoleBinding
- 元数据
    - HPA,PodTemplate,LimitRange,
- 

### 配置清单

什么是配置清单，举个例子来说，我们可以查看下目前集群中具有哪些pod。  

```shell
[root@k8s-master ~]# kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
nginx-deploy-5b595999-55ps9   1/1       Running   0          27m
nginx-deploy-5b595999-rgxlm   1/1       Running   0          27m
nginx-deploy-5b595999-xnwph   1/1       Running   0          27m
```
我们可以将其中一个pod的信息输出为`yaml` 格式。yaml格式的信息就是我们需要的配置清单。     


```shell
[root@k8s-master ~]# kubectl get pod nginx-deploy-5b595999-55ps9 -o yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: 2018-10-21T11:34:31Z
  generateName: nginx-deploy-5b595999-
  labels:
    pod-template-hash: "16151555"
    run: nginx-deploy
  name: nginx-deploy-5b595999-55ps9
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-deploy-5b595999
    uid: 95b4f426-d517-11e8-a995-000c296af098
  resourceVersion: "40728"
  selfLink: /api/v1/namespaces/default/pods/nginx-deploy-5b595999-55ps9
  uid: 46b95437-d525-11e8-be95-000c296af098
spec:
  containers:
  - image: nginx:1.14-alpine
    imagePullPolicy: IfNotPresent
    name: nginx-deploy
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-2wpbt
      readOnly: true
  dnsPolicy: ClusterFirst
  nodeName: k8s-node1
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-2wpbt
    secret:
      defaultMode: 420
      secretName: default-token-2wpbt
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: 2018-10-21T11:34:31Z
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: 2018-10-21T11:34:33Z
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: null
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: 2018-10-21T11:34:31Z
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://ebc9c81239fd9897bb3371743cc2240e8a5de503f7fc8d3c3513b401e1ed3091
    image: nginx:1.14-alpine
    imageID: docker-pullable://nginx@sha256:8976218be775f4244df2a60a169d44606b6978bac4375192074cefc0c7824ddf
    lastState: {}
    name: nginx-deploy
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: 2018-10-21T11:34:32Z
  hostIP: 192.168.0.40
  phase: Running
  podIP: 10.244.1.8
  qosClass: BestEffort
  startTime: 2018-10-21T11:34:31Z

```  

**创建资源的方法**:
-    apiserver仅接收JSON格式的资源定义;  
-    yaml格式提供配置清单,apiserver可自动将其转为json格式，而后再提交;  


**大部分资源的配置清单**:  

-    apiversion： 格式就是 group/version  

目前支持的apiversion，一共有下面这些类型。  
```shell
[root@k8s-master ~]# kubectl api-versions
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
apps/v1beta1
apps/v1beta2
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```

-    kind:资源类别,k8s支持的资源类型就是上一节我们介绍的那些。     

-    metadata：元数据
        - name：
        - namespace:
        - labels: 
        - annotations：
        - uid:
    
        每个资源的引用PATH：
            /api/GROUP/VERSION/namespaces/NAMESPACE/TYPE/NAME
-    spec： 定义用户期望的目标状态,disired state
-    status: 当前状态,current state,本字段由kubernetes集群维护;

**内建格式说明**  


    kubectl explain pods
    kubectl explain pods.metadata  

```shell

[root@k8s-master ~]# kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

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
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

```
    
只要能够看见对象的，就可以一级一级的向下查看

 
### 创建一个配置清单   

所有的列表都可以写成中括号  
所有的映射(键值)都可以写成大括号  
列表要加横线，键值不加横线  

```shell
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo 
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
  - name: busybox
    image: busybox:latest
    command: 
    - "/bin/sh"
    - "-c"
    - "date >> /usr/share/nginx/html/index.html; sleep 5"
```
### 创建pod  

```shell 
[root@k8s-master manifests]# kubectl create -f pod-demo.yaml
pod/pod-demo created
[root@k8s-master manifests]# kubectl get pods
NAME                          READY     STATUS              RESTARTS   AGE
nginx-deploy-5b595999-55ps9   1/1       Running             0          1h
nginx-deploy-5b595999-rgxlm   1/1       Running             0          1h
nginx-deploy-5b595999-xnwph   1/1       Running             0          1h
pod-demo                      0/2       ContainerCreating   0          41s
[root@k8s-master manifests]# kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
nginx-deploy-5b595999-55ps9   1/1       Running   0          1h
nginx-deploy-5b595999-rgxlm   1/1       Running   0          1h
nginx-deploy-5b595999-xnwph   1/1       Running   0          1h
pod-demo                      2/2       Running   2          1m
```

此时，我们去查看一下pod的详细信息，就可以发现，是我们自定义的清单信息。  

```shell
[root@k8s-master manifests]# kubectl describe pod pod-demo 
Name:               pod-demo
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node3/192.168.0.42
Start Time:         Sun, 21 Oct 2018 20:43:50 +0800
Labels:             app=myapp
                    tier=frontend
Annotations:        <none>
Status:             Running
IP:                 10.244.3.8
Containers:
  nginx:
    Container ID:   docker://1c6f212a7236248bb3436c32beda3c4df56939881533213748d7ea5e03bea61b
    Image:          nginx:1.13
    Image ID:       docker-pullable://nginx@sha256:b1d09e9718890e6ebbbd2bc319ef1611559e30ce1b6f56b2e3b479d9da51dc35
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 21 Oct 2018 20:44:35 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2wpbt (ro)
  busybox:
    Container ID:  docker://ad0688bc5f18b3e9bff94ccedbed91af1fb6f602bffdf2ba619ab5bff3b103c5
    Image:         busybox:latest
    Image ID:      docker-pullable://busybox@sha256:2a03a6059f21e150ae84b0973863609494aad70f0a80eaeb64bddd8d92465812
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      date >> /usr/share/nginx/html/index.html; sleep 5
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sun, 21 Oct 2018 20:45:45 +0800
      Finished:     Sun, 21 Oct 2018 20:45:50 +0800
    Ready:          False
    Restart Count:  3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2wpbt (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
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
  Type     Reason     Age               From                Message
  ----     ------     ----              ----                -------
  Normal   Scheduled  2m                default-scheduler   Successfully assigned default/pod-demo to k8s-node3
  Normal   Pulling    2m                kubelet, k8s-node3  pulling image "nginx:1.13"
  Normal   Pulled     1m                kubelet, k8s-node3  Successfully pulled image "nginx:1.13"
  Normal   Created    1m                kubelet, k8s-node3  Created container
  Normal   Started    1m                kubelet, k8s-node3  Started container
  Normal   Pulling    48s (x4 over 1m)  kubelet, k8s-node3  pulling image "busybox:latest"
  Normal   Pulled     46s (x4 over 1m)  kubelet, k8s-node3  Successfully pulled image "busybox:latest"
  Normal   Created    46s (x4 over 1m)  kubelet, k8s-node3  Created container
  Normal   Started    46s (x4 over 1m)  kubelet, k8s-node3  Started container
  Warning  BackOff    26s (x5 over 1m)  kubelet, k8s-node3  Back-off restarting failed container
```  

此时，nginx是能够进行访问到的，然后使用下面的命令就能够看到我们定义的容器的访问日志。    

```shell 
[root@k8s-master ~]# kubectl logs pod-demo nginx 
10.244.3.1 - - [21/Oct/2018:12:47:19 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
10.244.0.0 - - [21/Oct/2018:12:48:40 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
```


### 总结

上面的示例中，有一个命令是执行不成功的。那就是 ` date >> /usr/share/nginx/html/index.html; sleep 5 `,这是因为 busybox和nginx 分属于两个不同的镜像，他们呢的存储空间是不一样的。这个需要在后面我们接触了存储卷之后，再进行相应的完善。    


通过 `yaml` 清单文件创建的pod资源是可以删除的 直接使用 `kubectl delete pod [podname]` 就可以了。并且还可以根据我们自己的清单文件来进行删除，如下所示。  

```shell
[root@k8s-master manifests]# kubectl delete -f pod-demo.yaml 
pod "pod-demo" deleted
[root@k8s-master manifests]# kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
nginx-deploy-5b595999-55ps9   1/1       Running   0          1h
nginx-deploy-5b595999-rgxlm   1/1       Running   0          1h
nginx-deploy-5b595999-xnwph   1/1       Running   0          1h
```



