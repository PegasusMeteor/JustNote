## kubernetes ingress 以及ingress controller 

官方网站 [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)

### ingress 介绍

![nginx-ingress-controller](http://ot2trm1s2.bkt.clouddn.com/k8s/nginx-ingress-controller.png)

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

此时kubernetes会自动去下载镜像，并创建容器。使用下面的命令可以查看到相应的状态。  

```shell
[root@k8s-master ingress-nginx]# kubectl get pods -n ingress-nginx 
NAME                                        READY     STATUS              RESTARTS   AGE
nginx-ingress-controller-664f488479-9hczz   0/1       ContainerCreating   0          3m
```  
接下来，我们可以直接去创建ingress 了。    

为了更好的理解ingress ，我们首先创建几个后端被代理的 "主机"   




