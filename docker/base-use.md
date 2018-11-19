# Docker 基础用法

## Docker Engine 

Dokcer Engine 本身一个C/S架构的应用程序。包含下面的一些组件。   

![Docker Engine](http://www.selinux.tech/docker/engine-components-flow.png)  

## Docker Architecture  

![Docker Architecture](http://www.selinux.tech/docker/architecture.svg)  


### Docker daemon  

### Docker client  

### Docker registries


### Docker objects
When you use Docker, you are creating and using images, containers, networks, volumes, plugins, and other objects. This section is a brief overview of some of those objects.  

#### IMAGES  

#### CONTAINERS 

#### SERVICES  


## 安装Docker

Docker 有Docker-ee(企业版)和Docker-ce(社区版),我们直接安装docker-ce版本就可以了。  

可以直接从阿里云镜像站下载docker-ce 的yum源，直接使用yum安装就可以。 下载地址 [https://mirrors.aliyun.com/docker-ce/linux/centos/](https://mirrors.aliyun.com/docker-ce/linux/centos/)  

## Docker 镜像加速  

可以使用阿里云的镜像加速器，需要注册一个账号。阿里云的加速，对每个用户而言是不一样的，下面是我的加速设置。   

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://vbbcsxj5.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker
```  

## Docker 命令分组  

使用 `docker --help` 命令可以看到新版的docker已经对命令进行了分组，如下所示 .  

```shell
Management Commands:
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes  


```  

建议在以后的使用过程中使用最新的分组命令，例如创建容器有 `docker create  ... ` 与  `docker container create ...` 两种方式，建议使用后者，清晰明了。  


## Docker image 管理  

```shell 
~]# docker image --help 

Usage:  docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
```


## Docker Container 管理   

```shell
~]# docker container --help 

Usage:  docker container COMMAND

Manage containers

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  inspect     Display detailed information on one or more containers
  kill        Kill one or more running containers
  logs        Fetch the logs of a container
  ls          List containers
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  prune       Remove all stopped containers
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  run         Run a command in a new container
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker container COMMAND --help' for more information on a command.

```  

例如启动一个容器   

```shell
docker container run --name b1 -it busybox:latest
```  

## Docker 进入一个正在运行的容器  

```shell

docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]

```  

## Docker 查看容器日志   
Docker 容器在运行过程中出现问题的话，可以通过命令来查看docker 容器的运行错误 。
 
```shell
docker container logs [OPTIONS] CONTAINER
```  



## Docker Event State

![Docker Event State](http://www.selinux.tech/docker/event_state.png)










