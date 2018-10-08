RESTful
    GET,PUT,DELETE,POST
    kubectl run,get,edit

资源：对象
    - workload:Pod,ReplicaSet,Deployment,StatefulSet,DaemonSet,Job,CronJob,...
    - 服务发现及均衡:Service，Ingress
    - 配置与存储:Volume,CSI
        - ConfigMap,Secret
        - SownwardAPI
    - 集群级资源
        - Namespace,Node,Role,ClusterRole.RoleBinding,ClusterRoleBinding
    - 元数据
        - HPA,PodTemplate,LimitRange,
    - 


配置清单 yaml

举例来说

```shell
kubectl  get pod [podname] -o yaml

```  


创建资源的方法:
    apiserver仅接收JSON格式的资源定义;
    yaml格式提供配置清单,apiserver可自动将其转为json格式，而后再提交;


大部分资源的配置清单:
    apiversion： group/version
        $ kubectl api-versions

    
    kind:资源类别
    metadata：元数据
        name：
        namespace:
        labels:
        annotations：
        uid:
    
        每个资源的应用PATH
            /api/GROUP/VERSION/namespaces/NAMESPACE/TYPE/NAME
    spec： 期望的状态,disired state
    status: 当前状态,current state,本字段由kubernetes集群维护;


内建格式说明

    kubectl explain pods
    kubectl explain pods.metadata
    
    只要能够看见对象的，就可以一级一级的向下查看




创建一个清单示例
所有的列表都可以写成中括号
所有的键值都可以写成大括号
列表要加横线，键值不加横线


vim pod-demo.yaml

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
    - name: myapp
      image: ikubernetes/myapp:v1
    - name: busybox
      image: busybox:latest
      command: 
      - "/bin/sh"
      - "-c"
      - "sleep 3600"


创建pod

kubectl create -f pod-demo.yaml



