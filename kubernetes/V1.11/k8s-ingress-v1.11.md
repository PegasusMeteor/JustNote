## kubernetes ingress 以及ingress controller 

github地址 [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)   

官方地址 [https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)

### ingress 介绍

![nginx-ingress-controller](https://www.selinux.tech/k8s/nginx-ingress-controller.png)

**后期要补充完善**

```shell
[root@k8s-master ~]# kubectl explain ingress
KIND:     Ingress
VERSION:  extensions/v1beta1

DESCRIPTION:
     Ingress is a collection of rules that allow inbound connections to reach
     the endpoints defined by a backend. An Ingress can be configured to give
     services externally-reachable urls, load balance traffic, terminate SSL,
     offer name based virtual hosting etc.

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
     Spec is the desired state of the Ingress. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

   status       <Object>
     Status is the current state of the Ingress. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status


[root@k8s-master ~]# kubectl explain ingress.spec.rules
KIND:     Ingress
VERSION:  extensions/v1beta1

RESOURCE: rules <[]Object>

DESCRIPTION:
     A list of host rules used to configure the Ingress. If unspecified, or no
     rule matches, all traffic is sent to the default backend.

     IngressRule represents the rules mapping the paths under a specified host
     to the related backend services. Incoming requests are first evaluated for
     a host match, then routed to the backend associated with the matching
     IngressRuleValue.

FIELDS:
   host <string>
     Host is the fully qualified domain name of a network host, as defined by
     RFC 3986. Note the following deviations from the "host" part of the URI as
     defined in the RFC: 1. IPs are not allowed. Currently an IngressRuleValue
     can only apply to the IP in the Spec of the parent Ingress. 2. The `:`
     delimiter is not respected because ports are not allowed. Currently the
     port of an Ingress is implicitly :80 for http and :443 for https. Both
     these may change in the future. Incoming requests are matched against the
     host before the IngressRuleValue. If the host is unspecified, the Ingress
     routes all traffic based on the specified IngressRuleValue.

   http <Object>

```

### 部署一个ingress controller

ingress controller 作为一个核心附件而存在。  coreDNS 也是其中一个。

kubernetes/ingress-nginx 的 github 地址 [https://github.com/kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)   

将这个git库，下载到本地。然后将其 deploy 目录单独摘出来。然后，使用apply 来创建ingress  

```shell
[root@k8s-master ingress-nginx]# ll
total 24
-rw-r--r-- 1 root root  199 Nov  7 23:01 configmap.yaml
-rw-r--r-- 1 root root 5326 Nov  7 23:01 mandatory.yaml
-rw-r--r-- 1 root root   69 Nov  7 23:01 namespace.yaml
-rw-r--r-- 1 root root 2866 Nov  7 23:01 rbac.yaml
-rw-r--r-- 1 root root 2193 Nov  7 23:01 with-rbac.yaml


[root@k8s-master ingress-nginx]# kubectl apply -f namespace.yaml
.......
[root@k8s-master ingress-nginx]# kubectl apply -f ./
..........

```

此时kubernetes会自动去下载镜像，并创建容器。使用下面的命令可以查看到相应的状态。 其中 `ingress-nginx` 是由 `namespace.yaml`文件来创建的。

```shell
[root@k8s-master ingress-nginx]# kubectl get pods -n ingress-nginx 
NAME                                        READY     STATUS              RESTARTS   AGE
nginx-ingress-controller-664f488479-9hczz   0/1       ContainerCreating   0          3m
```  
容器创建的过程有些缓慢，需要稍等一下。或者是由于国内网络的原因，如果在之前使用docker的时候，设置了代理，那么这里是没有问题的。但是国内环境很多情况下，是不能够下载成功的。
如果下载不成功的话，可以去阿里云上寻找相应的镜像来代替。
可以使用 ` kubectl describe pods nginx-ingress-controller-664f488479-p7gdf -n ingress-nginx`


接下来，我们可以直接去创建ingress 了。    

### 创建ingress 

```shell
[root@k8s-master ~]# kubectl explain ingress.spec
KIND:     Ingress
VERSION:  extensions/v1beta1

RESOURCE: spec <Object>

DESCRIPTION:
     Spec is the desired state of the Ingress. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

     IngressSpec describes the Ingress the user wishes to exist.

FIELDS:
   backend      <Object>
     A default backend capable of servicing requests that don't match any rule.
     At least one of 'backend' or 'rules' must be specified. This field is
     optional to allow the loadbalancer controller or defaulting logic to
     specify a global default.

   rules        <[]Object>
     A list of host rules used to configure the Ingress. If unspecified, or no
     rule matches, all traffic is sent to the default backend.

   tls  <[]Object>
     TLS configuration. Currently the Ingress only supports a single TLS port,
     443. If multiple members of this list specify different hosts, they will be
     multiplexed on the same port according to the hostname specified through
     the SNI TLS extension, if the ingress controller fulfilling the ingress
     supports SNI.
```


为了更好的理解ingress ，我们首先创建几个后端被代理的 "主机"。

```shell
[root@k8s-master ingress]# cat deploy-demo.yaml 
apiVersion: v1
kind: Service
metadata: 
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  ports:
  - name: http
    targetPort: 80
    port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 3
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
        
[root@k8s-master ingress]# kubectl apply -f deploy-demo.yaml 
service/myapp created
deployment.apps/myapp-deploy created

[root@k8s-master ~]# kubectl get pods 
NAME                            READY     STATUS    RESTARTS   AGE
myapp-deploy-6d96b6cc45-4tk7h   1/1       Running   0          2m
myapp-deploy-6d96b6cc45-lz9f4   1/1       Running   0          2m
myapp-deploy-6d96b6cc45-mh2nk   1/1       Running   0          2m

[root@k8s-master ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    71d
myapp        ClusterIP   10.110.39.98   <none>        80/TCP     3m


```  

因为我们需要外部的流量进入到这个集群中，所以我们需要将ClusterIP 修改为NodePort. 官方介绍[https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal)    

我们可以将nodeport这个文件下载到本地，并修改nodePort为一个固定端口。避免端口随机。    

```shell
[root@k8s-master ingress]# wget  https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
...  ...  ...


[root@k8s-master ingress]# cat service-nodeport.yaml 
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
      nodePort: 30443
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

```   

接下来我们可以应用一下这个 `service-port` .

```shell

[root@k8s-master ingress]# kubectl apply -f service-nodeport.yaml 
service/ingress-nginx created
[root@k8s-master ingress]# kubectl get svc -n ingress-nginx
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.111.165.217   <none>        80:30080/TCP,443:30443/TCP   31s
```  

接下来，我们可以尝试在集群外部去访问一下我们的应用。   如果能够看到404就说明，我们是部署成功了的。  

### 针对我们后端的应用定义ingress 


```shell

[root@k8s-master ingress]# cat ingress-myapp.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default 
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec: 
  rules:
  - host: myapp.selinux.tech  # 这里要确保，在外部通过域名访问的时候能够访问到后端的service，需要修改hosts文件
    http:
      paths:
      - path:
        backend:
          serviceName: myapp
          servicePort: 80 

    
[root@k8s-master ingress]# kubectl apply -f ingress-myapp.yaml 
ingress.extensions/ingress-myapp created

```  

接下来就可以看看我们创建好的ingress 

```shell

[root@k8s-master ingress]# kubectl get ingress
NAME            HOSTS                ADDRESS   PORTS     AGE
ingress-myapp   myapp.selinux.tech             80        4m
[root@k8s-master ingress]# kubectl describe ingress ingress-myapp
Name:             ingress-myapp
Namespace:        default
Address:          
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                Path  Backends
  ----                ----  --------
  myapp.selinux.tech  
                         myapp:80 (<none>)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{"kubernetes.io/ingress.class":"nginx"},"name":"ingress-myapp","namespace":"default"},"spec":{"rules":[{"host":"myapp.selinux.tech","http":{"paths":[{"backend":{"serviceName":"myapp","servicePort":80},"path":null}]}}]}}

  kubernetes.io/ingress.class:  nginx
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  4m    nginx-ingress-controller  Ingress default/ingress-myapp
```  

此时，通过外部的浏览器来访问我们域名就可以访问到后端的应用。  

![https://www.selinux.tech/k8s/ingress_test.png](https://www.selinux.tech/k8s/ingress_test.png)  

同上，如果需要部署其他的应用，就按照这个过程来进行操作就可以了。  














