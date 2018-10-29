## kubernetes pod 控制器二


### 容器存活探测（exec）

```shell
[root@k8s-master ~]# kubectl explain pods.spec.containers.livenessProbe.exec
KIND:     Pod
VERSION:  v1

RESOURCE: exec <Object>

DESCRIPTION:
     One and only one of the following should be specified. Exec specifies the
     action to take.

     ExecAction describes a "run in container" action.

FIELDS:
   command      <[]string>
     Command is the command line to execute inside the container, the working
     directory for the command is root ('/') in the container's filesystem. The
     command is simply exec'd, it is not run inside a shell, so traditional
     shell instructions ('|', etc) won't work. To use a shell, you need to
     explicitly call out to that shell. Exit status of 0 is treated as
     live/healthy and non-zero is unhealthy.

```



前面 pod生命周期的介绍中，我们已经列举了一个探针示例。接下来我们再给出一个示例。  

```shell
[root@k8s-master manifests]# cat liveness-exec.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec-pod 
  namespace: default
spec:
  containers:
  - name: liveness-exec-container
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c","touch /tmp/healthy;sleep 30; rm -f /tmp/healthy;sleep 3600"]
    livenessProbe: 
      exec:
        command: ["test","-e","/tmp/healthy"]
      initialDelaySeconds: 2
      periodSeconds: 3
 #restartPolicy: OnFailure
```

运行一下上面的pod。过一段时间之后，查看一下pod的状态。  

```shell
[root@k8s-master manifests]# kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
liveness-exec-pod             1/1       Running   2          2m
nginx-deploy-5b595999-55ps9   1/1       Running   0          8d
nginx-deploy-5b595999-rgxlm   1/1       Running   0          8d
nginx-deploy-5b595999-xnwph   1/1       Running   0          8d
pod-demo                      0/2       Pending   0          2h
```
从上面可以看出，pod是被restart几次了。
这就是 pod的exec探测。

### 容器存活探测（tcpSocket）

接下来，介绍一下容器探测的 tcpSocket方式。
```shell
[root@k8s-master ~]# kubectl explain pods.spec.containers.livenessProbe.tcpSocket
KIND:     Pod
VERSION:  v1

RESOURCE: tcpSocket <Object>

DESCRIPTION:
     TCPSocket specifies an action involving a TCP port. TCP hooks not yet
     supported

     TCPSocketAction describes an action based on opening a socket

FIELDS:
   host <string>
     Optional: Host name to connect to, defaults to the pod IP.

   port <string> -required-
     Number or name of the port to access on the container. Number must be in
     the range 1 to 65535. Name must be an IANA_SVC_NAME.
```


### 容器存活探测（httpGet）


```shell
[root@k8s-master ~]# kubectl explain pods.spec.containers.livenessProbe.httpGet
KIND:     Pod
VERSION:  v1

RESOURCE: httpGet <Object>

DESCRIPTION:
     HTTPGet specifies the http request to perform.

     HTTPGetAction describes an action based on HTTP Get requests.

FIELDS:
   host <string>
     Host name to connect to, defaults to the pod IP. You probably want to set
     "Host" in httpHeaders instead.

   httpHeaders  <[]Object>
     Custom headers to set in the request. HTTP allows repeated headers.

   path <string>
     Path to access on the HTTP server.

   port <string> -required-
     Name or number of the port to access on the container. Number must be in
     the range 1 to 65535. Name must be an IANA_SVC_NAME.

   scheme       <string>
     Scheme to use for connecting to the host. Defaults to HTTP.
```

可以参考下面的示例。  

```shell
[root@k8s-master manifests]# cat liveness-httpget.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: liveness-httpget-pod 
  namespace: default
spec:
  containers:
  - name: liveness-httpget-container
    image: nginx:1.13
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80 
    livenessProbe: 
      httpGet:
        port: http
        path: /index.html
      initialDelaySeconds: 2
      periodSeconds: 3
 #restartPolicy: OnFailure
```

然后创建这个pod，并查看其运行状态。当这个pod运行起来之后，可以查看一下这个pod的描述信息。这时就会发现，liveness探测起到了作用。   

```shell
[root@k8s-master manifests]# kubectl  describe pod liveness-httpget-pod
Name:               liveness-httpget-pod
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node3/192.168.0.42
Start Time:         Mon, 29 Oct 2018 22:08:55 +0800
Labels:             <none>
Annotations:        <none>
Status:             Running
IP:                 10.244.3.14
Containers:
  liveness-httpget-container:
    Container ID:   docker://ccc910df9de478a23556a1738d7c0ee5a559a48988041625a761305b3a2aefe3
    Image:          nginx:1.13
    Image ID:       docker-pullable://nginx@sha256:b1d09e9718890e6ebbbd2bc319ef1611559e30ce1b6f56b2e3b479d9da51dc35
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 29 Oct 2018 22:08:56 +0800
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:http/index.html delay=2s timeout=1s period=3s #success=1 #failure=3
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
  Normal  Scheduled  1m    default-scheduler   Successfully assigned default/liveness-httpget-pod to k8s-node3
  Normal  Pulled     1m    kubelet, k8s-node3  Container image "nginx:1.13" already present on machine
  Normal  Created    1m    kubelet, k8s-node3  Created container
  Normal  Started    1m    kubelet, k8s-node3  Started container

```

如果我们连入 `liveness-httpget-pod` 容器，将 `/index.html` 删除掉，此时就很容器发现，pod会被重启一下。



### 容器就绪探测

`readinessProbe`的使用与`livenessProbe`的使用是一样的。

```shell
[root@k8s-master ~]# kubectl explain pods.spec.containers.readinessProbe
KIND:     Pod
VERSION:  v1

RESOURCE: readinessProbe <Object>

DESCRIPTION:
     Periodic probe of container service readiness. Container will be removed
     from service endpoints if the probe fails. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes

     Probe describes a health check to be performed against a container to
     determine whether it is alive or ready to receive traffic.

FIELDS:
   exec <Object>
     One and only one of the following should be specified. Exec specifies the
     action to take.

   failureThreshold     <integer>
     Minimum consecutive failures for the probe to be considered failed after
     having succeeded. Defaults to 3. Minimum value is 1.

   httpGet      <Object>
     HTTPGet specifies the http request to perform.

   initialDelaySeconds  <integer>
     Number of seconds after the container has started before liveness probes
     are initiated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes

   periodSeconds        <integer>
     How often (in seconds) to perform the probe. Default to 10 seconds. Minimum
     value is 1.

   successThreshold     <integer>
     Minimum consecutive successes for the probe to be considered successful
     after having failed. Defaults to 1. Must be 1 for liveness. Minimum value
     is 1.

   tcpSocket    <Object>
     TCPSocket specifies an action involving a TCP port. TCP hooks not yet
     supported

   timeoutSeconds       <integer>
     Number of seconds after which the probe times out. Defaults to 1 second.
     Minimum value is 1. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes

```

就绪探针的探测，会对容器进行就绪性检测，随时对容器进行判断，看其是否处于就绪状态。

```shell
[root@k8s-master manifests]# cat readiness-httpget.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: readiness-httpget-pod 
  namespace: default
spec:
  containers:
  - name: readiness-httpget-container
    image: nginx:1.13
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80 
    readinessProbe: 
      httpGet:
        port: http
        path: /index.html
      initialDelaySeconds: 2
      periodSeconds: 3
```
使用 ` kubectl create -f   readiness-httpget.yaml` 命令创建上面的pod,并查看pod的状态。 可以看到 readiness-httpget-pod 的READY状态为 `1/1`

```shell
[root@k8s-master ~]# kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
liveness-httpget-pod          1/1       Running   0          18m
nginx-deploy-5b595999-55ps9   1/1       Running   0          8d
nginx-deploy-5b595999-rgxlm   1/1       Running   0          8d
nginx-deploy-5b595999-xnwph   1/1       Running   0          8d
readiness-httpget-pod         1/1       Running   0          4m
``` 

此时我们通过 `kubectl exec -it readiness-httpget-pod -- /bin/sh` 命令链接进入容器，并将 index.html 文件删除掉，再查看pod状态的话，就会发现，虽然 readiness-httpget-pod 这个pod还在运行，但是其READY值已经变为了 `0/1` 了。

```shell
[root@k8s-master ~]# kubectl get pods 
NAME                          READY     STATUS    RESTARTS   AGE
liveness-httpget-pod          1/1       Running   0          20m
nginx-deploy-5b595999-55ps9   1/1       Running   0          8d
nginx-deploy-5b595999-rgxlm   1/1       Running   0          8d
nginx-deploy-5b595999-xnwph   1/1       Running   0          8d
readiness-httpget-pod         0/1       Running   0          6m
```

同理，进入重启，重新创建一个 index.html 文件,pod 的状态就会恢复正常了。当然也可以通过 `kubectl describe pod readiness-httpget-pod` 来查看健康状态探测的信息。  



### lifecycle 中的postStart 和 postStop

想要了解lifecycle,以及lifecycle中的postStart 和 postStop，就要对lifecycle的整个过程有一个比较详细的了解。以及postStart和postStop分别是在pod创建的哪个阶段执行。这些需要进行详细的分析和认识。下面给出一个简单的例子。  

```shell
[root@k8s-master manifests]# cat poststart-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: poststart-pod
  namespace: default
spec:
  containers:
  - name: busybox-httpd
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh","-c","mkdir -p /data/web/html; echo Home_Page >> /data/web/html/index.html"]
    command: ["/bin/sh"]
    args: ["-c","sleep 3600"]
```









