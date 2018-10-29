## kubernetes pod 控制器一


### Pod资源  

关于pod资源，上一节已经进行了介绍。这里我们再介绍一下 资源清单中 spec 字段中的资源。

Pod资源:  

    spec:
        containers <[]Object>   
            name         <string>
            image        <string>  
            imagePullPolicy        <string>  
                镜像的下载策略，Always, Never, IfNotPresent
            ports <[]Object>
                containerPort        <integer> -required-
                hostIP       <string>
                hostPort     <integer>
                name     <string>
            command      <[]string>
            args <[]string>
                command和args可以修改镜像中的默认应用
                https://v1-11.docs.kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/


### 标签的使用示例 

        kubectl get pods --help 

**`-l`,指定标签选择器，根据某个标签选择器来查看某个pod的标签**

```shell
[root@k8s-master manifests]# kubectl get pods -l app --show-labels
NAME       READY     STATUS    RESTARTS   AGE       LABELS
pod-demo   1/2       Running   0          18s       app=myapp,tier=frontend
```  

**查看所有pod的标签**

```shell
[root@k8s-master manifests]# kubectl get pods  --show-labels      
NAME                          READY     STATUS    RESTARTS   AGE       LABELS
nginx-deploy-5b595999-55ps9   1/1       Running   0          2h        pod-template-hash=16151555,run=nginx-deploy
nginx-deploy-5b595999-rgxlm   1/1       Running   0          2h        pod-template-hash=16151555,run=nginx-deploy
nginx-deploy-5b595999-xnwph   1/1       Running   0          2h        pod-template-hash=16151555,run=nginx-deploy
pod-demo                      1/2       Running   2          1m        app=myapp,tier=frontend
```   

**新增一列来显示标签，如果有相应的标签，这一列就会有值，可以指定多个 -L**.

```
[root@k8s-master manifests]# kubectl get pods  -L app
NAME                          READY     STATUS             RESTARTS   AGE       APP
nginx-deploy-5b595999-55ps9   1/1       Running            0          2h        
nginx-deploy-5b595999-rgxlm   1/1       Running            0          2h        
nginx-deploy-5b595999-xnwph   1/1       Running            0          2h        
pod-demo                      1/2       CrashLoopBackOff   3          2m        myapp
```
**新增一个新的标签**
```shell
[root@k8s-master manifests]# kubectl label pods pod-demo release=canary 
pod/pod-demo labeled
[root@k8s-master manifests]# kubectl  get pods -l app --show-labels
NAME       READY     STATUS             RESTARTS   AGE       LABELS
pod-demo   1/2       CrashLoopBackOff   5          10m       app=myapp,release=canary,tier=frontend
```

**如果原来的标签已经有值，那就需要进行覆盖**
```shell
[root@k8s-master manifests]# kubectl label pods pod-demo release=stable --overwrite 
pod/pod-demo labeled
[root@k8s-master manifests]# kubectl  get pods -l app --show-labels                 
NAME       READY     STATUS             RESTARTS   AGE       LABELS
pod-demo   1/2       CrashLoopBackOff   6          12m       app=myapp,release=stable,tier=frontend
```

### 标签选择器  

**等值关系： =,==,!=**  
**集合关系：**  
    &emsp;&emsp;KEY in (VALUE1,VALUE2,...)  
    &emsp;&emsp;KEY not in (VALUE1,VALUE2,...)  
    &emsp;&emsp;KEY  
    &emsp;&emsp;!KEY  

许多资源支持内嵌字段定义其使用的标签选择器：  
**matchLabels** ： 直接给定键值    
**matchExpressions** ： 基于给定的表达式来定义使用标签选择器,{key:"KEY",operator:"OPERATOR",values:[VAL1,VAL2,...]}  
&emsp;&emsp;操作符： 
&emsp;&emsp;&emsp;&emsp;In,NotIn:values 字段值，必须为非空列表。  
&emsp;&emsp;&emsp;&emsp;Exists,NotExists:values 字段值必须为空列表。  

查找标签等于某个值的pod

```shell
[root@k8s-master ~]# kubectl  get pods -l release=stable 
NAME       READY     STATUS             RESTARTS   AGE
pod-demo   1/2       CrashLoopBackOff   7          19m
```

根据集合关系来查找pod

```shell
[root@k8s-master ~]# kubectl  get pods -l "release in (canary,alpha,beta)"   
No resources found.
[root@k8s-master ~]# kubectl  get pods -l "release notin (canary,alpha,beta)"   
NAME                          READY     STATUS             RESTARTS   AGE
nginx-deploy-5b595999-55ps9   1/1       Running            0          3h
nginx-deploy-5b595999-rgxlm   1/1       Running            0          3h
nginx-deploy-5b595999-xnwph   1/1       Running            0          3h
pod-demo                      1/2       CrashLoopBackOff   7          21m
```

### 节点标签选择器  

继续以之前的pod-demo.yaml这清单为例。nodeSelector是与Container同级的一个资源标签。  

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
    - "echo $(date) >> /usr/share/nginx/html/index.html; sleep 5"
  nodeSelector: 
    disktype: ssd
```

然后执行下面的命令，重新创建一下这个pod-demo 。 


```shell
[root@k8s-master manifests]# kubectl delete -f pod-demo.yaml 
pod "pod-demo" deleted
[root@k8s-master manifests]# kubectl create -f pod-demo.yaml 
pod/pod-demo created
```
此时，再去查看一下pod-demo的信息就可以发现，新增了一个nodeSelector。

```shell
[root@k8s-master manifests]# kubectl describe pod  pod-demo           
Name:               pod-demo
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               <none>
Labels:             app=myapp
                    tier=frontend
Annotations:        <none>
Status:             Pending
IP:                 
Containers:
  nginx:
    Image:        nginx:1.13
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2wpbt (ro)
  busybox:
    Image:      busybox:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sh
      -c
      echo $(date) >> /usr/share/nginx/html/index.html; sleep 5
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2wpbt (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  default-token-2wpbt:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-2wpbt
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  disktype=ssd
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age               From               Message
  ----     ------            ----              ----               -------
  Warning  FailedScheduling  3m (x25 over 4m)  default-scheduler  0/4 nodes are available: 4 node(s) didn't match node selector.
```


除了节点标签选择器之外，还有如下一些重要的属性字段 nodeName,annoations。

### nodeName 

### annoations

与label 不同的地方在于，它不能用于挑选资源对象，仅用于为对象提供"元数据"。






