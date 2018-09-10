# Docker 存储卷

Docker 镜像 由多个只读层内叠加而成，启动容器时，docker会加载只读镜像层，并在镜像栈顶部添加一个读写层。

如果运行中的容器修改了现有的一个已经存在的文件，那么改为文件将会从读写层下面的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写层中该文件的副本所隐藏，此即下“写时复制(COW)”机制。


![docker-data-volumes](http://ot2trm1s2.bkt.clouddn.com/docker/docker-data-volumes.png)


# 为什么要用到存储卷

关闭并重启容器，其数据不受影响。但删除Docker容器，则其更改会全部丢失。

存在的问题:
- 存储于联合文件系统中，不易于宿主机访问；
- 容器见数据共享不便；
- 删除容器其数据会丢失；

解决方案:"卷"
    - "卷"是容器上的一个或者多个"目录"，此类目录可绕过联合文件系统，与宿主机上的某个目录"绑定"  



# Data volumes 

Docker 存储卷为持久数据或共享数据提供了一些有用的特性
- Volume 在容器初始化之时就会创建，由base images 提供的卷中的数据会在此期间完成复制。  
- Data volumes can be shared and reused among containers 
- 对数据卷的更改是直接进行的
- 升级docker image 的时候，对数据卷的数据没有影响
- 如果容器本身被删掉了，数据卷还会被保留
- Volumes are easier to back up or migrate than bind mounts.
- You can manage volumes using Docker CLI commands or the - Docker API.
- Volumes work on both Linux and Windows containers.
- Volumes can be more safely shared among multiple containers.
- Volume drivers let you store volumes on remote hosts or - cloud providers, to encrypt the contents of volumes, or to - add other functionality.
- New volumes can have their content pre-populated by a - container.  

Volume的初衷是独立于容器的生命周期实现数据持久化，因此，删除容器之时既不会删除卷，也不会对哪怕未被引用的卷做垃圾回收操作。  


卷为docker 提供了独立于容器的数据管理机制
- 可以把"镜像" 想象成静态文件，例如"程序",把卷类比为动态内容，例如"数据";于是可以重用，而卷可以共享。  
- 卷实现了"程序(镜像)"和"数据(卷)"分离，以及"程序(镜像)"和"制作镜像的主机"分离，用户制作时无须考虑镜像运行的容器所在的主机的环境。

# 卷类型 

docker 有两种类型的卷，每种类型都在容器中存在一个挂载点，但其在宿主机上的位置有所不同。  

- bind mount volume 
    在宿主机上，用户指定的挂载目录
- Docker-managed volume 
    docker deamon 来进行维护的，位于 /usr/lib/docker/[container_id] 
    将容器删除之后，卷会被删除掉。



# 在容器中使用Volumes

Docker run 命令使用-v 选项即可使用volumes
- Docker-managed volume 
```shell
    ~]# docker run -it --name volume-test1 -v /data busybox
    ~]# docker inspect -f {{.Mounts}}  volume-test1
```
- Bind-mount Volume  
```shell
     ~]# docker run -it -v HOSTDIR:VOLUMEDIR --name volume-test2 busybox
    ~]# docker inspect -f {{.Mounts}}  volume-test2
```

# 复制其他容器的存储卷设置(共享存储卷)

Docker 支持在启动一个容器的时候直接复制其他的容器的存储卷设置。
- 多个容器的卷使用同一个主机目录,例如

```shell
    ~]# docker run -it -v /docker/volume-test1:/data --name volume-test1 busybox
    ~]# docker run -it -v /docker/volume-test1:/data --name volume-test2 busybox
```

- 复制使用其他容器的卷,为docker run 命令使用 --volumes-from 选项

```shell
    ~]# docker run -it -v /docker/volume-test1:/data --name volume-test1 busybox
    ~]# docker run -it --name volume-test2 --volumes-from  volume-test1 busybox
```


![docker share volumes][http://ot2trm1s2.bkt.clouddn.com/docker/dcoker-share-volumes.png]()

