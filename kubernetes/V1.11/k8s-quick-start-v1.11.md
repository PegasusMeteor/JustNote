## kubernetes V1.11 应用快速入门



### kubectl 命令介绍

kubectl 就是apiserver 的客户端工具

kubectl 可以完成k8s各种资源的增删该查

例如下面的这些资源

pod,service,replicaset,deployment,statefulet,daemonset,job,cronjob,node


kubectl 有众多子命令

可以分为下面几类

```shell
kubectl ＋ tab 键就能够查看


```


### 运行nginx
介绍以下 kubectl run 命令
```
kubectl run --help 

```

启动一个nginx 实例

```
kubectl run nginx-deploy --image=nginx:1.14-alpine  --port=80 --replicas=1 --dry-run=true

```

查看验证

```
kubectl get deployment 

```

```
kubectl get pods 
kubectl get pods  -o wide 


```


访问  通过 pod 的网络地址
只能在集群节点内部进行访问

```

```

那如果外部类型的客户端想要访问的话，应该如何访问呢？
应该给pod 一个固定端点
这应该通过service 来实现
然后通过标签和标签选择器Selector 来访问pod




```
kubectl expose deployment nginx-deploy --name=nginx --port --target-port=80  --protocol=TCP 
```

```
kubectl get servies
kubectl get svc

```





扩展pod
动态扩容和缩容

```shell
kubectl scale --replicas=5 deployment myapp

```




升级版本

```
kubectl set image  

```



回滚

```shell
kubectl rollout undo 
```





