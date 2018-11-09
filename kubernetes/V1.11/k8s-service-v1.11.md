## kubernetes service 资源

官方文档 [https://kubernetes.io/zh/docs/concepts/services-networking/service/](https://kubernetes.io/zh/docs/concepts/services-networking/service/)

![userservice代理模式](http://ot2trm1s2.bkt.clouddn.com/k8s/services-userspace-overview.svg)

![iptables代理模式](http://ot2trm1s2.bkt.clouddn.com/k8s/services-iptables-overview.svg)



### Service 

工作模式： userspace,iptables,ipvs
    userspace: 1.1-
    iptables: 1.10-
    ipvs: 1.11+

类型:
    ExternalName,ClusterIP,NodePort,LoadBalancer


```shell
[root@k8s-master ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   65d
[root@k8s-master ~]# kubectl explain svc
KIND:     Service
VERSION:  v1

DESCRIPTION:
     Service is a named abstraction of software service (for example, mysql)
     consisting of local port (for example 3306) that the proxy listens on, and
     the selector that determines which pods will answer requests sent through
     the proxy.

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
     Spec defines the behavior of a service.
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the service. Populated by the system.
     Read-only. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status
```


接下来，介绍如何使用清单文件来创建service 资源


```shell
[root@k8s-master manifests]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   65d
[root@k8s-master manifests]# kubectl apply -f redis-svc.yaml 
service/redis created
[root@k8s-master manifests]# kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP    65d
redis        ClusterIP   10.97.97.97   <none>        6379/TCP   9s
[root@k8s-master manifests]# cat redis-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: default 
spec:
  selector:
    app: redis
    role: logstore
  clusterIP: 10.97.97.97
  type: ClusterIP
  ports: 
  - port: 6379 
    targetPort: 6379 
  

[root@k8s-master manifests]# kubectl describe svc redis
Name:              redis
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"redis","namespace":"default"},"spec":{"clusterIP":"10.97.97.97","ports":[{"por...
Selector:          app=redis,role=logstore
Type:              ClusterIP
IP:                10.97.97.97
Port:              <unset>  6379/TCP
TargetPort:        6379/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```

