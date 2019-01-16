# 容器网络  

![Docker容器网络的四种模式](https://www.selinux.tech/docker/docker-network.png)

```shell
 ~]# ip netns help 
Usage: ip netns list
       ip netns add NAME
       ip netns set NAME NETNSID
       ip [-all] netns delete [NAME]
       ip netns identify [PID]
       ip netns pids NAME
       ip [-all] netns exec [NAME] cmd ...
       ip netns monitor
       ip netns list-id
```

## Closed Container   

```shell
 ~]# docker run --name t1 -it --network none --rm busybox:latest
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
/ # ifconfig -a
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

## Bridged Container 


```shell

~]# docker run --name t1 -it --network bridge --rm busybox:latest
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:508 (508.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

## Joined Container   

联盟式容器是指，使用某个已存在的容器的网络接口的容器，接口被联盟内各容器共享使用。因为，联盟式容器批次之间完全无隔离，例如。
- 创建一个监听于2222端口的http服务器容器
```shell
 ~]# docker run -d -it --rm -p 2222 busybox:latest /bin/httpd -p 2222 -f
```  
- 创建一个联盟式容器，并查看起监听的端口

```shell
 ~]# docker run -it --rm --network  container:[container_name] --name joined busybox:latest netstat -tan 
```  

联盟式容器彼此之间虽然共享同一个网络名称空间，但是其他名称空间例如User,Mount等还是隔离的。  
联盟式容器彼此之间存在端口冲突的可能性，因为，通常只会在多个容器上的的程序需要通过loopback接口互相通信，或对某已经存在的容器的网络 属性进行监控时才使用此种模式的网络模型。  



## Open Container   

-p 选项的使用格式  
- -p containerPort 将指定的容器端口映射到主机所有地址的一个动态端口  
- -p hostPort:containerPort 将容器端口 containerPort映射至主机端口hostPort
- -p ip::containerPort 将指定的容器端口containerPort映射至主机指定ip的动态端口  
- -p ip:hostPort:containerPort 将指定的容器端口containerPort映射至主机指定的ip的端口hostPort 

"动态端口"指随机端口，具体的映射结果可以使用docker port命令来查看  

而所有的端口之间的转发规则，都是通过iptables转发规则来实现的。 


## deamon.json文件
 
dockerd守护进程的C/S，其默认仅监听Unix SOcket格式的地址，/var/run/docker.sock;如果使用TCP套接字，

        /etc/docker/daemon.json：
            "hosts": ["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"]
                
也可向dockerd直接传递“-H|--host”选项；
            
            
        
        
自定义docker0桥的网络属性信息：/etc/daemon.json文件
{
    "bip": "192.168.1.5/24",
    "fixed-cidr": "10.20.0.0/16",
    "fixed-cidr-v6": "2001:db8::/64",
    "mtu": 1500,
    "default-gateway": "10.20.1.1",
    "default-gateway-v6": "2001:db8:abcd::89",
    "dns": ["10.20.1.2","10.20.1.3"]
} 
核心选项为bip，即bridge ip 之意，用于指定docker0桥自身的IP地址;








