## kubernetes StatefulSet

1. 稳定且唯一的网络标识符;  
1. 稳定且持久的存储;  
1. 有序、平滑地部署和扩展;
1. 有序、平滑地删除和终止;
1. 有序地滚动更新;  

**继续补充**  

```shell
[root@k8s-master ~]# kubectl explain sts
KIND:     StatefulSet
VERSION:  apps/v1

DESCRIPTION:
     StatefulSet represents a set of pods with consistent identities. Identities
     are defined as: - Network: A single stable DNS and hostname. - Storage: As
     many VolumeClaims as requested. The StatefulSet guarantees that a given
     network identity will always map to the same storage identity.

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

   spec <Object>
     Spec defines the desired identities of pods in this set.

   status       <Object>
     Status is the current status of Pods in this StatefulSet. This data may be
     out of date by some window of time.
```



编写一个statefulset的清单文件。   

```shell

[root@k8s-master manifests]# cat stateful-demo.yaml
apiVersion: v1
kind: Service
metadata: 
  name: myapp
  labels:
    app: myapp
spec:
  ports:
  - port: 80 
    name: web
  clusterIP: None
  selector: 
    app: myapp-pod
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: myapp
spec: 
  serviceName: myapp
  replicas: 2
  selector:
    matchLabels:
      app: myapp-pod
  template:
    metadata:
      labels: 
        app: myapp-pod
    spec:
      containers:
      - name: myapp
        image: nginx:1.13
        ports:
        - containerPort: 80 
          name: web
        volumeMounts: 
        - name: myappdata
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: myappdata
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 2Gi
```

接下来，创建pv.  

```shell
[root@k8s-master volumes]# kubectl apply -f pv-demo.yaml 
persistentvolume/pv001 created
persistentvolume/pv002 created
persistentvolume/pv003 created
persistentvolume/pv004 created
persistentvolume/pv005 created
[root@k8s-master volumes]# kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
pv001     10Gi       RWO,RWX        Retain           Available                                      7s
pv002     5Gi        RWO,RWX        Retain           Available                                      7s
pv003     2Gi        RWO,RWX        Retain           Available                                      7s
pv004     20Gi       RWO,RWX        Retain           Available                                      7s
pv005     10Gi       RWO,RWX        Retain           Available   
```


statefulSet在定义的时候使用到了pv，所以在创建statefulSet的时候，pv的资源必须要提前准备好。   

```shell
[root@k8s-master manifests]# kubectl apply -f stateful-demo.yaml 
service/myapp unchanged
statefulset.apps/myapp created
[root@k8s-master ~]# kubectl get sts
NAME      DESIRED   CURRENT   AGE
myapp     2         2         2m
[root@k8s-master ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   77d
myapp        ClusterIP   None         <none>        80/TCP    5m
[root@k8s-master ~]# kubectl get pvc
NAME                STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
myappdata-myapp-0   Bound     pv003     2Gi        RWO,RWX                       3m
myappdata-myapp-1   Bound     pv002     5Gi        RWO,RWX                       2m
[root@k8s-master ~]# kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                       STORAGECLASS   REASON    AGE
pv001     10Gi       RWO,RWX        Retain           Available                                                        13m
pv002     5Gi        RWO,RWX        Retain           Bound       default/myappdata-myapp-1                            13m
pv003     2Gi        RWO,RWX        Retain           Bound       default/myappdata-myapp-0                            13m
pv004     20Gi       RWO,RWX        Retain           Available                                                        13m
pv005     10Gi       RWO,RWX        Retain           Available                                                        13m
[root@k8s-master ~]# kubectl get pds
error: the server doesn't have a resource type "pds"
[root@k8s-master manifests]# kubectl get pods
NAME       READY     STATUS    RESTARTS   AGE
myapp-0    1/1       Running   0          4m
myapp-1    1/1       Running   0          3m

```


### 滚动更新  

