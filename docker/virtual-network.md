# 容器虚拟化网络概述  

6种名称空间:UTS,User,Mount,IPC,Pid,Net   
docker 一共又四种网络模型  




## Overlay Network 
是建立在tunnel基础上的一种叠加网络，类似于GRE tunnel


## Docker Network 

```shell
 ~]# docker network ls 
NETWORK ID          NAME                DRIVER              SCOPE
2a6f392db673        bridge              bridge              local
61c315eff94f        host                host                local
4df4b263196b        none                null                local
```  

每个容器内部，都有6个独立而隔离的namespace  

