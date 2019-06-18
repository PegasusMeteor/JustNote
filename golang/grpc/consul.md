# Consul

微服务的注册和发现是整个微服务架构中至关重要的一个环节，网上也有很多的关于注册发现的文章，因此我们在这里并不会过多的介绍。

同样，提起微服务的注册发现，很多人第一时间就会想到ZooKeeper，ETCD,eureka,Consul等众多组件。由于目前团队使用的Consul，同时最新版本的Consul也添加了对ServiceMesh的支持,这为我们接下来的ServiceMesh研究有参考价值，所以我们gRPC系列的服务注册和发现就以consul为主来进行介绍。

关于consul的原理和实际使用操作等内容，不是我们这次介绍的重点。后面有时间会详细学习一下consul的原理，然后另开一篇文章  [Architechture Consul](../../architecture/consul.md) 来进行详细的原理介绍。

## Consul安装

为了便于测试，我们这里采用 consul on docker 的形式安装。可以参考 consul 在dockerhub上的文章 [Consul and Docker](https://hub.docker.com/_/consul)

**1、将image pull 下来,采用最新版本就可以。**

```shell
docker pull consul
```

**2、 启动一个consul server**

```shell
docker run -d --name=dev-consul -p 8500:8500 -e CONSUL_BIND_INTERFACE=eth0 consul
```

这将运行一个完全基于内存的Consul服务器代理，其默认采用桥接网络，并且不在主机上显示任何服务。目前我的测试采用这种方式，但是不建议在生产中使用，因为一旦容器重启，所有的数据都会丢失。

查看一下容器启动的端口,并且对外暴露了8500 端口

```shell
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                        NAMES
47b494323296        consul              "docker-entrypoint.s…"   11 seconds ago      Up 9 seconds        8300-8302/tcp, 8500/tcp, 8301-8302/udp, 8600/tcp, 8600/udp, 0.0.0.0:8500->8500/tcp   dev-consul
```

然后使用 http://host-ip:8500 就可以打开consul的UI界面了.

使用exec 命令可以进入到容器内部。

```shell
docker exec -it dev-consul bin/sh
```

使用下面的命令查看,consul在容器内绑定的地址。

```shell
$ docker exec -t dev-consul ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:22 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1716 (1.6 KiB)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:535 errors:0 dropped:0 overruns:0 frame:0
          TX packets:535 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:38800 (37.8 KiB)  TX bytes:38800 (37.8 KiB)
```

 例如，如果该服务器在内部地址172.17.0.2上运行，则可以通过启动另外两个实例并告诉它们加入第一个节点来运行三节点集群以进行开发。

**3、加入另外两个节点**


```shell

$ docker run -d --name=dev-consul1 -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2

$ docker run -d --name=dev-consul2 -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2

```

查看集群中的所有成员

```shell

$ docker exec -t dev-consul consul members
Node          Address          Status  Type    Build  Protocol  DC   Segment
3a993c913dfa  172.17.0.3:8301  alive   server  1.5.1  2         dc1  <all>
47b494323296  172.17.0.2:8301  alive   server  1.5.1  2         dc1  <all>
7bc5b37a4e1a  172.17.0.4:8301  alive   server  1.5.1  2         dc1  <all>
```

**4、客户端模式下运行Consul Agent**

```shell
docker run -d  --name=dev-consul-agent   -e CONSUL_CLIENT_INTERFACE=eth0 consul agent  -retry-join=172.17.0.2
```

通过查看一下 `docker logs` 可以知道，这个agent已经能够进行注册发现的代理了。

```shell
$ docker logs 7509e2b8a31a

==> Found address '172.17.0.5' for interface 'eth0', setting client option...
==> Starting Consul agent...
==> Consul agent running!
           Version: 'v1.5.1'
           Node ID: 'a10e290d-9d1f-69a0-dc19-58c08fe7eaf3'
         Node name: '7509e2b8a31a'
        Datacenter: 'dc1' (Segment: '')
            Server: false (Bootstrap: false)
       Client Addr: [172.17.0.5] (HTTP: 8500, HTTPS: -1, gRPC: -1, DNS: 8600)
      Cluster Addr: 172.17.0.5 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false

==> Log data will now stream in as it occurs:

    2019/06/18 07:58:00 [INFO] serf: EventMemberJoin: 7509e2b8a31a 172.17.0.5
    2019/06/18 07:58:00 [INFO] agent: Started DNS server 172.17.0.5:8600 (udp)
    2019/06/18 07:58:00 [INFO] agent: Started DNS server 172.17.0.5:8600 (tcp)
    2019/06/18 07:58:00 [INFO] agent: Started HTTP server on 172.17.0.5:8500 (tcp)
    2019/06/18 07:58:00 [INFO] agent: started state syncer
    2019/06/18 07:58:00 [INFO] serf: EventMemberJoin: 47b494323296 172.17.0.2
    2019/06/18 07:58:00 [INFO] agent: Retry join LAN is supported for: aliyun aws azure digitalocean gce k8s mdns os packet scaleway softlayer triton vsphere
    2019/06/18 07:58:00 [INFO] consul: adding server 47b494323296 (Addr: tcp/172.17.0.2:8300) (DC: dc1)
    2019/06/18 07:58:00 [INFO] agent: Joining LAN cluster...
    2019/06/18 07:58:00 [INFO] agent: (LAN) joining: [172.17.0.2]
    2019/06/18 07:58:00 [WARN] manager: No servers available
    2019/06/18 07:58:00 [ERR] agent: failed to sync remote state: No known Consul servers
    2019/06/18 07:58:00 [INFO] serf: EventMemberJoin: 3a993c913dfa 172.17.0.3
    2019/06/18 07:58:00 [INFO] serf: EventMemberJoin: 7bc5b37a4e1a 172.17.0.4
    2019/06/18 07:58:00 [INFO] consul: adding server 3a993c913dfa (Addr: tcp/172.17.0.3:8300) (DC: dc1)
    2019/06/18 07:58:00 [INFO] consul: adding server 7bc5b37a4e1a (Addr: tcp/172.17.0.4:8300) (DC: dc1)
    2019/06/18 07:58:00 [INFO] agent: (LAN) joined: 1 Err: <nil>
    2019/06/18 07:58:00 [INFO] agent: Join LAN completed. Synced with 1 initial agents
    2019/06/18 07:58:00 [INFO] agent: Synced node info
```

