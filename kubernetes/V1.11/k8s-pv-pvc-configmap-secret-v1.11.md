## kubernetes PV、PVC、configmap和secret


配置容器化应用的方式：
1. 自定义命令行参数; 例如args   
1. 把配置文件直接配置进入镜像
1. 环境变量 
    - Cloud Native 的应用程序一般可以直接通过环境变量加载配置;
    - 通过entrypoint脚本来预处理变量为配置文件中的配置信息;
1. 存储卷   



### configmap  

一个configmap就是一系列配置数据的集合，这些数据可以被注入到pod中的容器中进行使用。   

configmap 属于名称空间级别的资源。  

接下来，我们演示如何创建一个configmap。  


```shell
[root@k8s-master ~]# kubectl explain cm
KIND:     ConfigMap
VERSION:  v1

DESCRIPTION:
     ConfigMap holds configuration data for pods to consume.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   binaryData   <map[string]string>
     BinaryData contains the binary data. Each key must consist of alphanumeric
     characters, '-', '_' or '.'. BinaryData can contain byte sequences that are
     not in the UTF-8 range. The keys stored in BinaryData must not overlap with
     the ones in the Data field, this is enforced during validation process.
     Using this field will require 1.10+ apiserver and kubelet.

   data <map[string]string>
     Data contains the configuration data. Each key must consist of alphanumeric
     characters, '-', '_' or '.'. Values with non-UTF-8 byte sequences must use
     the BinaryData field. The keys stored in Data must not overlap with the
     keys in the BinaryData field, this is enforced during validation process.

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

```
由上面的介绍可以看出，configmap很简单。可以编写资源清单文件直接创建，当然如果只是一次性创建的话，也可以直接创建一个configmap用来临时使用。  

```shell
[root@k8s-master ~]# kubectl create configmap --help 
Create a configmap based on a file, directory, or specified literal value. 

A single configmap may package one or more key/value pairs. 

When creating a configmap based on a file, the key will default to the basename of the file, and the
value will default to the file content.  If the basename is an invalid key, you may specify an
alternate key. 

When creating a configmap based on a directory, each file whose basename is a valid key in the
directory will be packaged into the configmap.  Any directory entries except regular files are
ignored (e.g. subdirectories, symlinks, devices, pipes, etc).

Aliases:
configmap, cm

Examples:
  # Create a new configmap named my-config based on folder bar
  kubectl create configmap my-config --from-file=path/to/bar
  
  # Create a new configmap named my-config with specified keys instead of file basenames on disk
  kubectl create configmap my-config --from-file=key1=/path/to/bar/file1.txt
--from-file=key2=/path/to/bar/file2.txt
  
  # Create a new configmap named my-config with key1=config1 and key2=config2
  kubectl create configmap my-config --from-literal=key1=config1 --from-literal=key2=config2
  
  # Create a new configmap named my-config from the key=value pairs in the file
  kubectl create configmap my-config --from-file=path/to/bar
  
  # Create a new configmap named my-config from an env file
  kubectl create configmap my-config --from-env-file=path/to/bar.env

Options:
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or
map key is missing in the template. Only applies to golang and jsonpath output formats.
      --append-hash=false: Append a hash of the configmap to its name.
      --dry-run=false: If true, only print the object that would be sent, without sending it.
      --from-env-file='': Specify the path to a file to read lines of key=val pairs to create a
configmap (i.e. a Docker .env file).
      --from-file=[]: Key file can be specified using its file path, in which case file basename
will be used as configmap key, or optionally with a key and file path, in which case the given key
will be used.  Specifying a directory will iterate each named file in the directory whose basename
is a valid configmap key.
      --from-literal=[]: Specify a key and literal value to insert in configmap (i.e.
mykey=somevalue)
      --generator='configmap/v1': The name of the API generator to use.
  -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|templatefile|template|jsonpath|jsonpath-file.
      --save-config=false: If true, the configuration of current object will be saved in its
annotation. Otherwise, the annotation will be unchanged. This flag is useful when you want to
perform kubectl apply on this object in the future.
      --template='': Template string or path to template file to use when -o=go-template,
-o=go-template-file. The template format is golang templates
[http://golang.org/pkg/text/template/#pkg-overview].
      --validate=true: If true, use a schema to validate the input before sending it

Usage:
  kubectl create configmap NAME [--from-file=[key=]source] [--from-literal=key1=value1] [--dry-run]
[options]

Use "kubectl options" for a list of global command-line options (applies to all com
```
#### 通过命令行创建
接下来，我们就直接在命令行中创建一个configmap  

```shell
[root@k8s-master ~]# kubectl create configmap nginx-config --from-literal=nginx_port=80 --from-literal=server_name=selinux.tech
configmap/nginx-config created
[root@k8s-master ~]# kubectl get cm
NAME           DATA      AGE
nginx-config   2         5s
[root@k8s-master ~]# kubectl describe cm nginx-config 
Name:         nginx-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx_port:
----
80
server_name:
----
selinux.tech
Events:  <none>
```

另外一种创建方式 ,首先编辑一个配置文件，以下面的配置文件为例。  
```shell
[root@k8s-master configmap]# cat www.conf 
server {
        server_name selinux.tech
        listen 80:
        root /data/web/html
}
```
接下来，通过指定文件的形式创建一个configmap   

```shell
[root@k8s-master configmap]# kubectl create configmap nginx-www --from-file=./www.conf 
configmap/nginx-www created
[root@k8s-master configmap]# kubectl get cm 
NAME           DATA      AGE
nginx-config   2         3m
nginx-www      1         8s
[root@k8s-master configmap]# kubectl get cm nginx-www -o yaml 
apiVersion: v1
data:
  www.conf: "server {\n\tserver_name selinux.tech\n\tlisten 80:\n\troot /data/web/html\n}\n\n"
kind: ConfigMap
metadata:
  creationTimestamp: 2018-11-14T13:47:39Z
  name: nginx-www
  namespace: default
  resourceVersion: "188235"
  selfLink: /api/v1/namespaces/default/configmaps/nginx-www
  uid: da3ba501-e813-11e8-bcc2-000c296af098

[root@k8s-master configmap]# kubectl describe cm nginx-www
Name:         nginx-www
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
www.conf:
----
server {
  server_name selinux.tech
  listen 80:
  root /data/web/html
}


Events:  <none>
```

通过 describe 和 -o yaml 的形式都能够查看configmap 



#### 通过清单文件创建  

接下来，我们根据前面创建的两个configmap，来创建一个pod，并在pod中应用前面我们创建的两个cm。  


```shell
[root@k8s-master configmap]# cat pod-configmap.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-1
  namespace: default
  labels:
    app: myapp
    tier: frontend 
  annotations:
    xiaoshuaigege.github.io/created-by: "cluster admin" 
spec:
  containers:
  - name: nginx
    image: nginx:1.13
    ports:     
    - name: http
      containerPort: 80 
    env: 
    - name: NGINX_SERVER_PORT 
      valueFrom:
        configMapKeyRef:
          name: nginx-config
          key: nginx_port 
    - name: NGINX_SERVER_NAME
      valueFrom: 
        configMapKeyRef:
          name: nginx-config
          key: server_name
```
接下来，我们创建这个pod，并进去这个pod查看我们定义好的环境变量。  

```shell
[root@k8s-master configmap]# kubectl apply -f pod-configmap.yaml 
pod/pod-cm-1 created
[root@k8s-master configmap]# kubectl get pods
NAME       READY     STATUS    RESTARTS   AGE
pod-cm-1   1/1       Running   0          11s
[root@k8s-master configmap]# kubectl exec -it pod-cm-1 -- /bin/sh 
# printenv       
KUBERNETES_PORT=tcp://10.96.0.1:443
MYAPP_SERVICE_PORT_HTTP=80
KUBERNETES_SERVICE_PORT=443
HOSTNAME=pod-cm-1
HOME=/root
MYAPP_SERVICE_HOST=10.110.39.98
NGINX_SERVER_PORT=80
NGINX_SERVER_NAME=selinux.tech
MYAPP_SERVICE_PORT=80
MYAPP_PORT=tcp://10.110.39.98:80
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NGINX_VERSION=1.13.12-1~stretch
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MYAPP_PORT_80_TCP_ADDR=10.110.39.98
KUBERNETES_PORT_443_TCP_PORT=443
NJS_VERSION=1.13.12.0.2.0-1~stretch
KUBERNETES_PORT_443_TCP_PROTO=tcp
MYAPP_PORT_80_TCP_PORT=80
MYAPP_PORT_80_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
MYAPP_PORT_80_TCP=tcp://10.110.39.98:80
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
#

```

从上面可以看到，pod的容器中已经包含了我们创建的两个env环境变量。  


#### 通过存储卷的方式访问configmap  


```shell
[root@k8s-master configmap]# cat pod-configmap-2.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-2
  namespace: default
  labels:
    app: myapp
    tier: frontend 
  annotations:
    xiaoshuaigege.github.io/created-by: "cluster admin" 
spec:
  containers:
  - name: nginx
    image: nginx:1.13
    ports:     
    - name: http
      containerPort: 80 
    volumeMounts:
    - name: nginxconf
      mountPath: /etc/nginx/config.d/
      readOnly: true
  volumes:
  - name: nginxconf
    configMap:
      name: nginx-config

[root@k8s-master configmap]# kubectl apply -f pod-configmap-2.yaml
pod/pod-cm-2 created
[root@k8s-master configmap]# kubectl get pods 
NAME       READY     STATUS    RESTARTS   AGE
pod-cm-2   1/1       Running   0          7s
```

接下来，我们连入这个刚刚创建的pod，进入到挂载目录下，查看一下是否有我们在configmap中创建的配置信息。   

```shell
[root@k8s-master configmap]# kubectl exec -it pod-cm-2 -- /bin/sh
# cd /etc/nginx/config.d
# ls
nginx_port  server_name
# cat server_name       
selinux.tech# 
```  

接下来，我们进行这样的一个实验，修改一下configmap中的值，然后再进入到pod中查看，看看挂载卷目录下的文件中值是否发生了变化。    

使用下面的命令，可以直接编辑修改 configmap。
```shell
[root@k8s-master configmap]# kubectl edit cm nginx-config
[root@k8s-master configmap]# kubectl describe cm nginx-config
Name:         nginx-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx_port:
----
8080
server_name:
----
selinux.tech
Events:  <none>
```

可以看到，与之前的定义相比，我们将端口换成了8080.然后我们再连入pod-cm-2 去查看一下，挂载目录下的文件是否发生了变化。  

```shell
[root@k8s-master configmap]# kubectl exec -it pod-cm-2 -- /bin/sh
# cd /etc/nginx/config.d
# ls
nginx_port  server_name
# cat server_name
selinux.tech
#        
# cat nginx_port  
8080
# 
```   
从上面可以看出，端口已经变成了8080.   
因此，在实际使用中，我们可以通过在外部创建配置文件，然后注入到pod中的方式来更改一个应用的配置。   



### secert   

什么是secert?   

```shell
[root@k8s-master ~]# kubectl create secret --help 
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry
  generic         Create a secret from a local file, directory or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```  

接下来，我们创建一个secert  

```shell
[root@k8s-master ~]# kubectl create secret generic mysql-root-password --from-literal=password=hello123
secret/mysql-root-password created
[root@k8s-master ~]# kubectl get secret
NAME                  TYPE                                  DATA      AGE
default-token-2wpbt   kubernetes.io/service-account-token   3         76d
mysql-root-password   Opaque 
```  

此时，我们去查看一下这个secert    

```shell
[root@k8s-master ~]# kubectl describe secret mysql-root-password
Name:         mysql-root-password
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes
```
此时，会显示，password这个key，它的值是8个字节，但是具体是什么内容不会显示出来。   

那我们通过 -o yaml 的方式去查看，会不会显示呢？  

```shell
[root@k8s-master ~]# kubectl get secret mysql-root-password -o yaml
apiVersion: v1
data:
  password: aGVsbG8xMjM=
kind: Secret
metadata:
  creationTimestamp: 2018-11-14T14:30:59Z
  name: mysql-root-password
  namespace: default
  resourceVersion: "192504"
  selfLink: /api/v1/namespaces/default/secrets/mysql-root-password
  uid: e796fd09-e819-11e8-bcc2-000c296af098
type: Opaque
```

还是没有显示，并且应该已经被加密了。使用的是base64编码。如果想要解密的话，使用base64解码就可以了。   


















