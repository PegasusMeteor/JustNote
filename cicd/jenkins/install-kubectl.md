# 安装Kubectl

## 参考

- [在 Linux 系统中安装并设置 kubectl](https://kubernetes.io/zh/docs/tasks/tools/install-kubectl-linux/)


## 安装kubectl {#install-kubectl-on-linux}


### 用 curl 在 Linux 系统中安装 kubectl {#install-kubectl-binary-with-curl-on-linux}

<!-- 
1. Download the latest release with the command:
-->
1. 用以下命令下载最新发行版：

   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   ```

   {% hint style="success" %}

   如需下载某个指定的版本，请用指定版本号替换该命令的这一部分：
   `$(curl -L -s https://dl.k8s.io/release/stable.txt)`。

   例如，要在 Linux 中下载 v1.22.0 版本，请输入：

   ```bash
   curl -LO https://dl.k8s.io/release/v1.22.0/bin/linux/amd64/kubectl
   ```
   {% endhint %}


2. 验证该可执行文件（可选步骤）

   下载 kubectl 校验和文件：

   ```bash
   curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
   ```

   基于校验和文件，验证 kubectl 的可执行文件：

   ```bash
   echo "$(<kubectl.sha256) kubectl" | sha256sum --check
   ```

   验证通过时，输出为：

   ```console
   kubectl: OK
   ```

   验证失败时，`sha256` 将以非零值退出，并打印如下输出：

   ```bash
   kubectl: FAILED
   sha256sum: WARNING: 1 computed checksum did NOT match
   ```

  {% hint style="success" %}

   下载的 kubectl 与校验和文件版本必须相同。
   {% endhint %}


3. 安装 kubectl

   ```bash
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   ```

   {% hint style="success" %}

   即使你没有目标系统的 root 权限，仍然可以将 kubectl 安装到目录 `~/.local/bin` 中：

   ```bash
   chmod +x kubectl
   mkdir -p ~/.local/bin/kubectl
   mv ./kubectl ~/.local/bin/kubectl
   # 之后将 ~/.local/bin/kubectl 添加到 $PATH
   ```
   {% endhint %}


4. 执行测试，以保障你安装的版本是最新的：

   ```bash
   kubectl version --client
   ```


## 配置访问k8s集群

将目标K8S集群中，Master节点上的 `/root/.kube/config` 文件拷贝到jenkins主机目录下。
这时，jenkins 就可以直接控制 目标K8S 集群了。
