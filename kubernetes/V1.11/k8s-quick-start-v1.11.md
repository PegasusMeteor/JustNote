## kubernetes V1.11 应用快速入门



### kubectl 命令介绍

kubectl 就是apiserver 的客户端工具。
kubectl 可以完成k8s各种资源的增删改查。

例如下面的这些资源
```
pod,service,replicaset,deployment,statefulet,daemonset,job,cronjob,node

```

kubectl 有众多子命令,可以分为下面几类

```shell
[root@k8s-master ~]# kubectl 
kubectl controls the Kubernetes cluster manager. 

Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create         Create a resource from a file or from stdin.
  expose         Take a replication controller, service, deployment or pod and expose it as a new
Kubernetes Service
  run            Run a particular image on the cluster
  set            Set specific features on objects

Basic Commands (Intermediate):
  explain        Documentation of resources
  get            Display one or many resources
  edit           Edit a resource on the server
  delete         Delete resources by filenames, stdin, resources and names, or by resources and
label selector

Deploy Commands:
  rollout        Manage the rollout of a resource
  scale          Set a new size for a Deployment, ReplicaSet, Replication Controller, or Job
  autoscale      Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:
  certificate    Modify certificate resources.
  cluster-info   Display cluster info
  top            Display Resource (CPU/Memory/Storage) usage.
  cordon         Mark node as unschedulable
  uncordon       Mark node as schedulable
  drain          Drain node in preparation for maintenance
  taint          Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe       Show details of a specific resource or group of resources
  logs           Print the logs for a container in a pod
  attach         Attach to a running container
  exec           Execute a command in a container
  port-forward   Forward one or more local ports to a pod
  proxy          Run a proxy to the Kubernetes API server
  cp             Copy files and directories to and from containers.
  auth           Inspect authorization

Advanced Commands:
  apply          Apply a configuration to a resource by filename or stdin
  patch          Update field(s) of a resource using strategic merge patch
  replace        Replace a resource by filename or stdin
  wait           Experimental: Wait for one condition on one or many resources
  convert        Convert config files between different API versions

Settings Commands:
  label          Update the labels on a resource
  annotate       Update the annotations on a resource
  completion     Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  alpha          Commands for features in alpha
  api-resources  Print the supported API resources on the server
  api-versions   Print the supported API versions on the server, in the form of "group/version"
  config         Modify kubeconfig files
  plugin         Runs a command-line plugin
  version        Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

```

例如如果查看k8s集群的信息的话，可以使用下面的命令。

```shell
[root@k8s-master ~]# kubectl cluster-info 
Kubernetes master is running at https://192.168.0.39:6443
KubeDNS is running at https://192.168.0.39:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```


### 以运行nginx为例

介绍以下 kubectl run 命令，run 命令中详细介绍了创建一个nginx pod过程。
```
[root@k8s-master ~]# kubectl run --help
Create and run a particular image, possibly replicated. 

Creates a deployment or job to manage the created container(s).

Examples:
  # Start a single instance of nginx.
  kubectl run nginx --image=nginx
  
  # Start a single instance of hazelcast and let the container expose port 5701 .
  kubectl run hazelcast --image=hazelcast --port=5701
  
  # Start a single instance of hazelcast and set environment variables "DNS_DOMAIN=cluster" and
"POD_NAMESPACE=default" in the container.
  kubectl run hazelcast --image=hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"
  
  # Start a single instance of hazelcast and set labels "app=hazelcast" and "env=prod" in the
container.
  kubectl run hazelcast --image=nginx --labels="app=hazelcast,env=prod"
  
  # Start a replicated instance of nginx.
  kubectl run nginx --image=nginx --replicas=5
  
  # Dry run. Print the corresponding API objects without creating them.
  kubectl run nginx --image=nginx --dry-run
  
  # Start a single instance of nginx, but overload the spec of the deployment with a partial set of
values parsed from JSON.
  kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'
  
  # Start a pod of busybox and keep it in the foreground, don't restart it if it exits.
  kubectl run -i -t busybox --image=busybox --restart=Never
  
  # Start the nginx container using the default command, but use custom arguments (arg1 .. argN) for
that command.
  kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>
  
  # Start the nginx container using a different command and custom arguments.
  kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>
  
  # Start the perl container to compute π to 2000 places and print it out.
  kubectl run pi --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'
  
  # Start the cron job to compute π to 2000 places and print it out every 5 minutes.
  kubectl run pi --schedule="0/5 * * * ?" --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle
'print bpi(2000)'

Options:
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or
map key is missing in the template. Only applies to golang and jsonpath output formats.
      --attach=false: If true, wait for the Pod to start running, and then attach to the Pod as if
'kubectl attach ...' were called.  Default false, unless '-i/--stdin' is set, in which case the
default is true. With '--restart=Never' the exit code of the container process is returned.
      --cascade=true: If true, cascade the deletion of the resources managed by this resource (e.g.
Pods created by a ReplicationController).  Default true.
      --command=false: If true and extra arguments are present, use them as the 'command' field in
the container, rather than the 'args' field which is the default.
      --dry-run=false: If true, only print the object that would be sent, without sending it.
      --env=[]: Environment variables to set in the container
      --expose=false: If true, a public, external service is created for the container(s) which are
run
  -f, --filename=[]: to use to replace the resource.
      --force=false: Only used when grace-period=0. If true, immediately remove resources from API
and bypass graceful deletion. Note that immediate deletion of some resources may result in
inconsistency or data loss and requires confirmation.
      --generator='': The name of the API generator to use, see
http://kubernetes.io/docs/user-guide/kubectl-conventions/#generators for a list.
      --grace-period=-1: Period of time in seconds given to the resource to terminate gracefully.
Ignored if negative. Set to 1 for immediate shutdown. Can only be set to 0 when --force is true
(force deletion).
      --hostport=-1: The host port mapping for the container port. To demonstrate a single-machine
container.
      --image='': The image for the container to run.
      --image-pull-policy='': The image pull policy for the container. If left empty, this value
will not be specified by the client and defaulted by the server
  -l, --labels='': Comma separated labels to apply to the pod(s). Will override previous values.
      --leave-stdin-open=false: If the pod is started in interactive mode or with stdin, leave stdin
open after the first attach completes. By default, stdin will be closed after the first attach
completes.
      --limits='': The resource requirement limits for this container.  For example,
'cpu=200m,memory=512Mi'.  Note that server side components may assign limits depending on the server
configuration, such as limit ranges.
  -o, --output='': Output format. One of:
json|yaml|name|go-template-file|templatefile|template|go-template|jsonpath|jsonpath-file.
      --overrides='': An inline JSON override for the generated object. If this is non-empty, it is
used to override the generated object. Requires that the object supply a valid apiVersion field.
      --pod-running-timeout=1m0s: The length of time (like 5s, 2m, or 3h, higher than zero) to wait
until at least one pod is running
      --port='': The port that this container exposes.  If --expose is true, this is also the port
used by the service that is created.
      --quiet=false: If true, suppress prompt messages.
      --record=false: Record current kubectl command in the resource annotation. If set to false, do
not record the command. If set to true, record the command. If not set, default to updating the
existing annotation value only if one already exists.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you
want to manage related manifests organized within the same directory.
  -r, --replicas=1: Number of replicas to create for this container. Default is 1.
      --requests='': The resource requirement requests for this container.  For example,
'cpu=100m,memory=256Mi'.  Note that server side components may assign requests depending on the
server configuration, such as limit ranges.
      --restart='Always': The restart policy for this Pod.  Legal values [Always, OnFailure, Never].
If set to 'Always' a deployment is created, if set to 'OnFailure' a job is created, if set to
'Never', a regular pod is created. For the latter two --replicas must be 1.  Default 'Always', for
CronJobs `Never`.
      --rm=false: If true, delete resources created in this command for attached containers.
      --save-config=false: If true, the configuration of current object will be saved in its
annotation. Otherwise, the annotation will be unchanged. This flag is useful when you want to
perform kubectl apply on this object in the future.
      --schedule='': A schedule in the Cron format the job should be run with.
      --service-generator='service/v2': The name of the generator to use for creating a service.
Only used if --expose is true
      --service-overrides='': An inline JSON override for the generated service object. If this is
non-empty, it is used to override the generated object. Requires that the object supply a valid
apiVersion field.  Only used if --expose is true.
      --serviceaccount='': Service account to set in the pod spec
  -i, --stdin=false: Keep stdin open on the container(s) in the pod, even if nothing is attached.
      --template='': Template string or path to template file to use when -o=go-template,
-o=go-template-file. The template format is golang templates
[http://golang.org/pkg/text/template/#pkg-overview].
      --timeout=0s: The length of time to wait before giving up on a delete, zero means determine a
timeout from the size of the object
  -t, --tty=false: Allocated a TTY for each container in the pod.
      --wait=false: If true, wait for resources to be gone before returning. This waits for
finalizers.

Usage:
  kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas]
[--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).

```

下面我们来启动一个nginx 实例,可以使用 `kubectl get deployment ` 来查看验证。

```

[root@k8s-master ~]# kubectl run nginx-deploy --image=nginx:1.14-alpine  --port=80 --replicas=1 --dry-run=true
deployment.apps/nginx-deploy created (dry run)
[root@k8s-master ~]# kubectl get deployment
No resources found.


## 下面我我们将--dry-run=true 删掉


[root@k8s-master ~]# kubectl run nginx-deploy --image=nginx:1.14-alpine  --port=80 --replicas=1
deployment.apps/nginx-deploy created
[root@k8s-master ~]# kubectl get deployment
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1         1         1            0           4s


```

下面来查看nginx 运行在哪个节点上，加上 `-o wide ` 显示详细信息。
```
[root@k8s-master ~]# kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
nginx-deploy-5b595999-tpw62   1/1       Running   0          24s
[root@k8s-master ~]# kubectl get pods -o wide
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE        NOMINATED NODE
nginx-deploy-5b595999-tpw62   1/1       Running   0          27s       10.244.3.6   k8s-node3   <none>


```

下面我们来访问一下试试，目前只能通过 pod 的网络地址，在集群节点内部进行访问。

```
[root@k8s-node3 ~]# curl 10.244.3.6
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

那如果外部类型的客户端想要访问的话，应该如何访问呢？或者如果这个pod被删除掉了，网络地址发生变化，我们如何访问呢？  
这样的话应该给pod一个固定端点，这应该通过service 来实现  
然后通过标签和标签选择器Selector 来访问pod。  

k8s expose 一个service的时候，也可以通过命令来查看相应的教程。  

```
[root@k8s-master ~]# kubectl expose --help 
Expose a resource as a new Kubernetes service. 

Looks up a deployment, service, replica set, replication controller or pod by name and uses the
selector for that resource as the selector for a new service on the specified port. A deployment or
replica set will be exposed as a service only if its selector is convertible to a selector that
service supports, i.e. when the selector contains only the matchLabels component. Note that if no
port is specified via --port and the exposed resource has multiple ports, all will be re-used by the
new service. Also if no labels are specified, the new service will re-use the labels from the
resource it exposes. 

Possible resources include (case insensitive): 

pod (po), service (svc), replicationcontroller (rc), deployment (deploy), replicaset (rs)

Examples:
  # Create a service for a replicated nginx, which serves on port 80 and connects to the containers
on port 8000.
  kubectl expose rc nginx --port=80 --target-port=8000
  
  # Create a service for a replication controller identified by type and name specified in
"nginx-controller.yaml", which serves on port 80 and connects to the containers on port 8000.
  kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000
  
  # Create a service for a pod valid-pod, which serves on port 444 with the name "frontend"
  kubectl expose pod valid-pod --port=444 --name=frontend
  
  # Create a second service based on the above service, exposing the container port 8443 as port 443
with the name "nginx-https"
  kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https
  
  # Create a service for a replicated streaming application on port 4100 balancing UDP traffic and
named 'video-stream'.
  kubectl expose rc streamer --port=4100 --protocol=udp --name=video-stream
  
  # Create a service for a replicated nginx using replica set, which serves on port 80 and connects
to the containers on port 8000.
  kubectl expose rs nginx --port=80 --target-port=8000
  
  # Create a service for an nginx deployment, which serves on port 80 and connects to the containers
on port 8000.
  kubectl expose deployment nginx --port=80 --target-port=8000

Options:
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or
map key is missing in the template. Only applies to golang and jsonpath output formats.
      --cluster-ip='': ClusterIP to be assigned to the service. Leave empty to auto-allocate, or set
to 'None' to create a headless service.
      --dry-run=false: If true, only print the object that would be sent, without sending it.
      --external-ip='': Additional external IP address (not managed by Kubernetes) to accept for the
service. If this IP is routed to a node, the service can be accessed by this IP in addition to its
generated service IP.
  -f, --filename=[]: Filename, directory, or URL to files identifying the resource to expose a
service
      --generator='service/v2': The name of the API generator to use. There are 2 generators:
'service/v1' and 'service/v2'. The only difference between them is that service port in v1 is named
'default', while it is left unnamed in v2. Default is 'service/v2'.
  -l, --labels='': Labels to apply to the service created by this call.
      --load-balancer-ip='': IP to assign to the LoadBalancer. If empty, an ephemeral IP will be
created and used (cloud-provider specific).
      --name='': The name for the newly created object.
  -o, --output='': Output format. One of:
json|yaml|name|templatefile|template|go-template|go-template-file|jsonpath|jsonpath-file.
      --overrides='': An inline JSON override for the generated object. If this is non-empty, it is
used to override the generated object. Requires that the object supply a valid apiVersion field.
      --port='': The port that the service should serve on. Copied from the resource being exposed,
if unspecified
      --protocol='': The network protocol for the service to be created. Default is 'TCP'.
      --record=false: Record current kubectl command in the resource annotation. If set to false, do
not record the command. If set to true, record the command. If not set, default to updating the
existing annotation value only if one already exists.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you
want to manage related manifests organized within the same directory.
      --save-config=false: If true, the configuration of current object will be saved in its
annotation. Otherwise, the annotation will be unchanged. This flag is useful when you want to
perform kubectl apply on this object in the future.
      --selector='': A label selector to use for this service. Only equality-based selector
requirements are supported. If empty (the default) infer the selector from the replication
controller or replica set.)
      --session-affinity='': If non-empty, set the session affinity for the service to this; legal
values: 'None', 'ClientIP'
      --target-port='': Name or number for the port on the container that the service should direct
traffic to. Optional.
      --template='': Template string or path to template file to use when -o=go-template,
-o=go-template-file. The template format is golang templates
[http://golang.org/pkg/text/template/#pkg-overview].
      --type='': Type for this service: ClusterIP, NodePort, LoadBalancer, or ExternalName. Default
is 'ClusterIP'.

Usage:
  kubectl expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP]
[--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type]
[options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

下面我们就来发布一个service 

```
[root@k8s-master ~]# kubectl expose deployment nginx-deploy --name=nginx --port=80  --target-port=80  --protocol=TCP 
service/nginx exposed

```
下面我们可以来查看目前发布的service

```
[root@k8s-master ~]# kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   52d
nginx        ClusterIP   10.103.67.25   <none>        80/TCP    24s
[root@k8s-master ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   52d
nginx        ClusterIP   10.103.67.25   <none>        80/TCP    32s

```

这样的话，我们在集群内部的节点就可以通过这个service来访问nginx了。  

但是目前这个nginx 还是不能够被外部网络的终端进行访问，虽然目前集群内部的所有节点都能够访问到。因为我们看到，虽然已经已经创建了service并给pod提供了一个固定的访问端点，但是service绑定的却是内部网络的地址。此时如果pod被重建了，service的地址也不会发生变化。这就是service的重要作用。


```shell
[root@k8s-master ~]# kubectl describe svc nginx
Name:              nginx
Namespace:         default
Labels:            run=nginx-deploy
Annotations:       <none>
Selector:          run=nginx-deploy
Type:              ClusterIP
IP:                10.103.67.25
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.3.6:80
Session Affinity:  None
Events:            <none>
```

根据上面的命令我们可以看出，不管nginx pod 被删除和重建多少次，service都可以根据 label  selector 找到 nginx-deploy 。此时通过 `kubectl describe svc [Name]` 来查看的时候，Endpoints 地址可能会发生变化，但是service 的内容却不会发生变化。  

但是如果service 被删除掉了，或者重建了，service 的地址就会发生变化了。


我们也可以查看 我们发布的pods 的详细信息。  
```shell
[root@k8s-master ~]# kubectl describe deployment nginx-deploy 
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Sun, 21 Oct 2018 17:56:30 +0800
Labels:                 run=nginx-deploy
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=nginx-deploy
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=nginx-deploy
  Containers:
   nginx-deploy:
    Image:        nginx:1.14-alpine
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deploy-5b595999 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  1h    deployment-controller  Scaled up replica set nginx-deploy-5b595999 to 1
```


### 外部访问service 

经过上面一系列的操作，我们还是不能够在别的网络中访问nginx,如果我们就是有这样的需求应该怎么办呢?其实很简单，只要修改nginx对应的service的配置就可以了.   
修改nginx 的service，将type由 `ClusterIP`修改为`NodePort`就可以了，注意大小写。   


```shell
[root@k8s-master ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   52d
nginx        ClusterIP   10.103.67.25   <none>        80/TCP    1h
[root@k8s-master ~]# kubectl edit svc nginx
service/nginx edited
[root@k8s-master ~]# kubectl describe svc nginx
Name:                     nginx
Namespace:                default
Labels:                   run=nginx-deploy
Annotations:              <none>
Selector:                 run=nginx-deploy
Type:                     NodePort
IP:                       10.103.67.25
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32579/TCP
Endpoints:                10.244.1.8:80,10.244.1.9:80,10.244.2.6:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
[root@k8s-master ~]# kubectl get  svc nginx        
NAME      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx     NodePort   10.103.67.25   <none>        80:32579/TCP   1h
```

这样的话，我们就可以通过集群中的任何一个节点的IP地址加上端口，来访问nginx的内容了。   

```
http://192.168.0.39:32579/
http://192.168.0.40:32579/
http://192.168.0.41:32579/
http://192.168.0.42:32579/

```

此时再配合负载均衡，就可以完成我们的需求了。  



### 扩展pod
我们程序的规模，以及有几个pod，是可以动态变动的。也就是动态扩容和缩容。  

这个命令就是 `kubectl scale `  

```shell
[root@k8s-master ~]# kubectl scale --help 
Set a new size for a Deployment, ReplicaSet, Replication Controller, or StatefulSet. 

Scale also allows users to specify one or more preconditions for the scale action. 

If --current-replicas or --resource-version is specified, it is validated before the scale is
attempted, and it is guaranteed that the precondition holds true when the scale is sent to the
server.

Examples:
  # Scale a replicaset named 'foo' to 3.
  kubectl scale --replicas=3 rs/foo
  
  # Scale a resource identified by type and name specified in "foo.yaml" to 3.
  kubectl scale --replicas=3 -f foo.yaml
  
  # If the deployment named mysql's current size is 2, scale mysql to 3.
  kubectl scale --current-replicas=2 --replicas=3 deployment/mysql
  
  # Scale multiple replication controllers.
  kubectl scale --replicas=5 rc/foo rc/bar rc/baz
  
  # Scale statefulset named 'web' to 3.
  kubectl scale --replicas=3 statefulset/web

Options:
      --all=false: Select all resources in the namespace of the specified resource types
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or
map key is missing in the template. Only applies to golang and jsonpath output formats.
      --current-replicas=-1: Precondition for current size. Requires that the current size of the
resource match this value in order to scale.
  -f, --filename=[]: Filename, directory, or URL to files identifying the resource to set a new size
  -o, --output='': Output format. One of:
json|yaml|name|go-template-file|templatefile|template|go-template|jsonpath|jsonpath-file.
      --record=false: Record current kubectl command in the resource annotation. If set to false, do
not record the command. If set to true, record the command. If not set, default to updating the
existing annotation value only if one already exists.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you
want to manage related manifests organized within the same directory.
      --replicas=0: The new desired number of replicas. Required.
      --resource-version='': Precondition for resource version. Requires that the current resource
version match this value in order to scale.
  -l, --selector='': Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l
key1=value1,key2=value2)
      --template='': Template string or path to template file to use when -o=go-template,
-o=go-template-file. The template format is golang templates
[http://golang.org/pkg/text/template/#pkg-overview].
      --timeout=0s: The length of time to wait before giving up on a scale operation, zero means
don't wait. Any other values should contain a corresponding time unit (e.g. 1s, 2m, 3h).

Usage:
  kubectl scale [--resource-version=version] [--current-replicas=count] --replicas=COUNT (-f
FILENAME | TYPE NAME) [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```  

下面我们对nginx进行扩展，扩展到5个pod。  

```shell
[root@k8s-master ~]# kubectl scale --replicas=5 deployment nginx-deploy
deployment.extensions/nginx-deploy scaled

[root@k8s-master ~]# kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
nginx-deploy-5b595999-2cnmr   1/1       Running   0          27s
nginx-deploy-5b595999-k9xsm   1/1       Running   0          27s
nginx-deploy-5b595999-kfljm   1/1       Running   0          27s
nginx-deploy-5b595999-tpw62   1/1       Running   0          40m
nginx-deploy-5b595999-xcvrs   1/1       Running   0          27s


[root@k8s-master ~]# kubectl get pods -o wide
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE        NOMINATED NODE
nginx-deploy-5b595999-2cnmr   1/1       Running   0          1m        10.244.1.5   k8s-node1   <none>
nginx-deploy-5b595999-k9xsm   1/1       Running   0          1m        10.244.3.7   k8s-node3   <none>
nginx-deploy-5b595999-kfljm   1/1       Running   0          1m        10.244.1.4   k8s-node1   <none>
nginx-deploy-5b595999-tpw62   1/1       Running   0          41m       10.244.3.6   k8s-node3   <none>
nginx-deploy-5b595999-xcvrs   1/1       Running   0          1m        10.244.2.4   k8s-node2   <none>
```

能够扩展，也就能够缩减，同样的，缩减pod资源非常简单，只要指定我们需要的pod副本数量就可以了。  
下面是缩容的过程。   

```shell
[root@k8s-master ~]# kubectl scale --replicas=3 deployment nginx-deploy 
deployment.extensions/nginx-deploy scaled
[root@k8s-master ~]# kubectl get pods -o wide                          
NAME                          READY     STATUS        RESTARTS   AGE       IP           NODE        NOMINATED NODE
nginx-deploy-5b595999-2cnmr   1/1       Terminating   0          4m        10.244.1.5   k8s-node1   <none>
nginx-deploy-5b595999-k9xsm   1/1       Running       0          4m        10.244.3.7   k8s-node3   <none>
nginx-deploy-5b595999-kfljm   1/1       Running       0          4m        10.244.1.4   k8s-node1   <none>
nginx-deploy-5b595999-tpw62   1/1       Running       0          44m       10.244.3.6   k8s-node3   <none>
nginx-deploy-5b595999-xcvrs   1/1       Terminating   0          4m        10.244.2.4   k8s-node2   <none>
[root@k8s-master ~]# kubectl get pods -o wide
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE        NOMINATED NODE
nginx-deploy-5b595999-k9xsm   1/1       Running   0          4m        10.244.3.7   k8s-node3   <none>
nginx-deploy-5b595999-kfljm   1/1       Running   0          4m        10.244.1.4   k8s-node1   <none>
nginx-deploy-5b595999-tpw62   1/1       Running   0          44m       10.244.3.6   k8s-node3   <none>
```

### 升级/降级版本

k8s 还可以对容器的版本进行升级或者降级。注意是容器，不是pod。所以在升级的时候，需要指定pod中的哪个容器。  
我们首先来查看一下当前的nginx是哪个版本。  

```shell
[root@k8s-master ~]# kubectl describe deployment   nginx-deploy         
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Sun, 21 Oct 2018 17:56:30 +0800
Labels:                 run=nginx-deploy
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=nginx-deploy
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=nginx-deploy
  Containers:
   nginx-deploy:
    Image:        nginx:1.14-alpine
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deploy-5b595999 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  1h    deployment-controller  Scaled up replica set nginx-deploy-5b595999 to 1
  Normal  ScalingReplicaSet  9m    deployment-controller  Scaled up replica set nginx-deploy-5b595999 to 5
  Normal  ScalingReplicaSet  4m    deployment-controller  Scaled down replica set nginx-deploy-5b595999 to 3
```

下面我们使用`set image `的方式来指定image的版本，从而进行升级和降级。

```
[root@k8s-master ~]# kubectl set image deployment  nginx-deploy nginx-deploy=nginx:1.13  
deployment.extensions/nginx-deploy image updated

```

此时再来查看一下nginx的版本会发现已经进行了降级。而且k8s在进行升级或者降级的时候，会逐个进行，灰度进行，保证正常的服务不会受到影响。   


```shell
[root@k8s-master ~]# kubectl describe deployment   nginx-deploy                         
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Sun, 21 Oct 2018 17:56:30 +0800
Labels:                 run=nginx-deploy
Annotations:            deployment.kubernetes.io/revision=2
Selector:               run=nginx-deploy
Replicas:               3 desired | 1 updated | 4 total | 3 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=nginx-deploy
  Containers:
   nginx-deploy:
    Image:        nginx:1.13
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  nginx-deploy-5b595999 (3/3 replicas created)
NewReplicaSet:   nginx-deploy-b6659bbdb (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  20m   deployment-controller  Scaled up replica set nginx-deploy-5b595999 to 5
  Normal  ScalingReplicaSet  16m   deployment-controller  Scaled down replica set nginx-deploy-5b595999 to 3
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deploy-b6659bbdb to 1
```

此时，再去 `kubectl get pods ` 就会发现，所有的pod已经发生变化了，因为被重建了。  

### 回滚
当然，能进行升级，就能够进行回滚，接下来我们对刚才进行的操作进行回滚。   

```shell 
[root@k8s-master ~]# kubectl rollout undo --help 
Rollback to a previous rollout.

Examples:
  # Rollback to the previous deployment
  kubectl rollout undo deployment/abc
  
  # Rollback to daemonset revision 3
  kubectl rollout undo daemonset/abc --to-revision=3
  
  # Rollback to the previous deployment with dry-run
  kubectl rollout undo --dry-run=true deployment/abc

Options:
      --dry-run=false: If true, only print the object that would be sent, without sending it.
  -f, --filename=[]: Filename, directory, or URL to files identifying the resource to get from a
server.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you
want to manage related manifests organized within the same directory.
      --to-revision=0: The revision to rollback to. Default to 0 (last revision).

Usage:
  kubectl rollout undo (TYPE NAME | TYPE/NAME) [flags] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).

```

此时我们就可以指明将谁 `undo` 到哪个版本，如果不指明，默认恢复到上一个版本。  
我们执行下面的命令，就可以看到，正在停止和回滚到上一个版本。  

```shell
[root@k8s-master ~]# kubectl rollout undo deployment nginx-deploy 
deployment.extensions/nginx-deploy
[root@k8s-master ~]# kubectl get pods -o wide
NAME                           READY     STATUS        RESTARTS   AGE       IP           NODE        NOMINATED NODE
nginx-deploy-5b595999-55ps9    1/1       Running       0          5s        10.244.1.8   k8s-node1   <none>
nginx-deploy-5b595999-rgxlm    1/1       Running       0          3s        10.244.1.9   k8s-node1   <none>
nginx-deploy-5b595999-xnwph    1/1       Running       0          7s        10.244.2.6   k8s-node2   <none>
nginx-deploy-b6659bbdb-86tkv   1/1       Terminating   0          10m       10.244.2.5   k8s-node2   <none>
nginx-deploy-b6659bbdb-v2m2d   0/1       Terminating   0          10m       10.244.1.6   k8s-node1   <none>
```  

此时，如果再去 `describe` 一下的话，会发现，版本已经进行了回滚。


## 总结

上面我们介绍的方式，只是最简单，最基本的一种使用K8S的方式。实际生产中，是不会这么简单的来使用K8S的。我们需要编写yaml配置文件，来进行各种配置。所以后面我会学习更高级的使用。








