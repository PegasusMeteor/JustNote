

1、关闭防火墙

2、关闭selinux

3、给master 和node 添加主机名解析，确保通过主机名能够访问到
172.20.10.2 k8s-1
172.20.10.3 k8s-2
172.20.10.5 k8s-3
172.20.10.4 master



4、确保时间同步

生产中一定要有时间服务器



timedatectl set-timezone Asia/Shanghai & systemctl restart chronyd.service



软件版本是截止笔者操作的时间版本





5、 在每个节点上部署docker

使用阿里云的docker-ce

https://mirrors.aliyun.com/docker-ce/linux/centos/



版本信息

Client:
 Version:           18.06.0-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        0ffa825
 Built:             Wed Jul 18 19:08:18 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.0-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       0ffa825
  Built:            Wed Jul 18 19:10:42 2018
  OS/Arch:          linux/amd64
  Experimental:     false




如果 k8s 运行在物理机上的话，master节点是不需要部署docker的。

yum install docker-ce -y

还需要配置 EPEL源，以及extras 

可以调查一下这里需要那些依赖。




docker 加速 需要注册阿里云 开发者平台
介绍一下阿里云的加速设置

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://vbbcsxj5.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

{
  "registry-mirrors": ["https://vbbcsxj5.mirror.aliyuncs.com"]
}


配置开机启动，并启动docker 


systemctl start docker & systemctl enable docker








k8s 安装
  官方介绍
  使用kubeadm 安装集群的方式
  https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl




  kubernetes rom repo:

  官方站点

    https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

  可以制作repo

  
  然后 阿里云 也有 yum的 路径

  https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/


]# yum info kubelet
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Available Packages
Name        : kubelet
Arch        : x86_64
Version     : 1.11.2
Release     : 0
Size        : 18 M
Repo        : name
Summary     : Container cluster management
URL         : https://kubernetes.io
License     : ASL 2.0
Description : The node agent of Kubernetes, the container cluster manager.



k8s 在github 上有托管站点


yum install kubeadm

master 上安装 kubeadm

其他节点主机上安装kubeadm 和kubelet

 yum install kubeadm kubelet

yum install kubelet 



rpm -ql kubeadm


修改 kubeadm 的dgrou-driver  
修改为与安装的docker一致
最新版本的kubenetes 已经发生了变化，会自动根据docker中运行的cgroup-driver进行配置变更，这个在官方网站上已经介绍了。


近用 swap ，官方建议禁用








