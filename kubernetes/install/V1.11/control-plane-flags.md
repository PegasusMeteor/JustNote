
## 使用kubeadm自定义控制台配置


 kubeadm configuration  开放了例如 APIServer, ControllerManager 和 Scheduler 等字段，通过这些字段可以修改控制台组件的一些默认参数:

- `APIServerExtraArgs`
- `ControllerManagerExtraArgs`
- `SchedulerExtraArgs`

这些字段是 `key: value` 键值对. 要override 控制台组件的默认参数的话，需要:

1.  将适当的字段添加到配置中.
2.  对字段添加相应的值.

对每个字段的配置细节可以移步[API reference pages](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#MasterConfiguration).



## APIServer 配置

点击 [reference documentation for kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)查看更多细节.

参考示例:
```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
metadata:
  name: 1.11-sample
apiServerExtraArgs:
  advertise-address: 192.168.0.103
  anonymous-auth: false
  enable-admission-plugins: AlwaysPullImages,DefaultStorageClass
  audit-log-path: /home/johndoe/audit.log
```

## ControllerManager 配置

点击 [reference documentation for kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)查看更多细节.

参考示例:
```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
metadata:
  name: 1.11-sample
controllerManagerExtraArgs:
  cluster-signing-key-file: /home/johndoe/keys/ca.key
  bind-address: 0.0.0.0
  deployment-controller-sync-period: 50
```

## Scheduler 配置

点击 [reference documentation for kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)查看更多细节.

参考示例:
```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
metadata:
  name: 1.11-sample
schedulerExtraArgs:
  address: 0.0.0.0
  config: /home/johndoe/schedconfig.yaml
  kubeconfig: /home/johndoe/kubeconfig.yaml
```

