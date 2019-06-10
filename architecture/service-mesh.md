# ServiceMesh 下一代微服务
<!-- TOC -->

- [ServiceMesh 下一代微服务](#servicemesh-%E4%B8%8B%E4%B8%80%E4%BB%A3%E5%BE%AE%E6%9C%8D%E5%8A%A1)
  - [What's a service mesh](#whats-a-service-mesh)
  - [Linkerd](#linkerd)
    - [Control Plane](#control-plane)
    - [DataPlane](#dataplane)
    - [Proxy](#proxy)
    - [CLI](#cli)
    - [Dashboard](#dashboard)
    - [Grafana](#grafana)
    - [Prometheus](#prometheus)
  - [Linkerd Or Istio](#linkerd-or-istio)
  - [参考](#%E5%8F%82%E8%80%83)

<!-- /TOC -->
## What's a service mesh

Service Mesh 这个概念最早由开发Linkerd 的 Buoyant, Inc 公司提出,而 Service Mesh 这个概念的定义则是 Buoyant, Inc 公司的 CEO William Morgan 于 2017 年 4 月 25 日 在 公司官网发布的的题 为 ["What's a service mesh? And why do I need one?"](https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/) 文章中给出的。

下面我们来看一下定义的内容。

> WHAT IS A SERVICE MESH?

> A service mesh is a dedicated infrastructure layer for handling service-to-service communication. It’s responsible for the reliable delivery of requests through the complex topology of services that comprise a modern, cloud native application. In practice, the service mesh is typically implemented as an array of lightweight network proxies that are deployed alongside application code, without the application needing to be aware. (But there are variations to this idea, as we’ll see.)

翻译过来就是：Service Mesh是用于处理服务与服务间通信的专用基础结构层。 云原生应用有复杂的服务拓扑图，Service Mesh可以保证请求在拓扑之间可靠地传递。 在实际应用中，Service Mesh 通常是由一系列轻量级的网络代理组成的，它们与应用程序部署在一起，但是应用程序并不需要知道他们的存在。

其实总结一下，无非就是下面两点：

1. Service Mesh 是一个专门负责请求可靠传输的基础架构层
2. Service Mesh 与应用部署在一起，通过网络代理来实现，应用程序无感知。

同时，William Morgan 对 Service Mesh 的概念进行了补充说明，明确了Service Mesh 的职责边界：

> The concept of the service mesh as a separate layer is tied to the rise of the cloud native application. In the cloud native model, a single application might consist of hundreds of services; each service might have thousands of instances; and each of those instances might be in a constantly-changing state as they are dynamically scheduled by an orchestrator like Kubernetes. Not only is service communication in this world incredibly complex, it’s a pervasive and fundamental part of runtime behavior. Managing it is vital to ensuring end-to-end performance and reliability.

Service Mesh 作为一个独立的基础架构层,是与云原生应用的崛起紧密相关的。在云原生模型里，一个单独的应用可能包含成百上千个服务，每个服务又可能包含成千上万个实例，每个实例可能处于不断变化的状态，因为有kubernetes这样类似的调度器存在。服务间通信不仅异常复杂，而且也是运行时行为的基础。管理好服务间通信对于保证端到端的性能和可靠性来说是非常重要的。

在笔者的理解中，Service Mesh 像本节的标题一样，是下一代微服务，它是一种架构，一种不是三言两语就能将其梳理清除新型架构。因此,只有结合实际的Service Mesh应用才能将其完全的理解。而目前在Service Mesh生态中，除了kubernetes，还有两个非常重要的组件。

接下来我们介绍一下ServiceMesh生态中两个非常重要的成员 Linkerd和Istio，通过对这两个成员的架构学习，尝试去搞清楚到 **What is Service Mesh?**.

## Linkerd

Linkerd 是一种Service Mesh (基于William Morgan的定义，因为就是他们公司的产品，所以它当然是一种Service Mesh)，它为云原生应用程序增加了可观察性，可靠性和安全性，无需更改代码。例如，Linkerd可以监控和报告每个服务的成功率和延迟，可以自动重试失败的请求，并且可以加密和验证服务之间的连接（TLS），所有这些都不需要对应用程序本身进行任何修改。

从较高层面来说，Linkerd 由 `control plane`和`data plane`组成。

`control plane` 是一组运行在特定 `namespace` (默认为linkerd)中的服务.这些服务完成各种诸如 聚合遥测数据，提供面向用户的API，向 `data plane` 代理发送控制数据等操作.这些操作共同驱动 `data plane`的行为。

`data plane`在是由与每个服务实例运行在一起的透明代理(服务无感知)组成。这些代理自动处理进出服务的所有流量。由于这些代理是透明的，所以他们充当了向 `control plane` 发送监测数据，并且从`control plane`接收控制数据的进程外控制程序。

![architecture （出处https://linkerd.io/images/architecture/control-plane.png）](images/control-plane.png)

### Control Plane

Linkerd `control plane` 是一组运行在特定 `namespace` (默认为linkerd)中的服务.这些服务完成各种诸如 聚合遥测数据，提供面向用户的API，向 `data plane` 代理发送控制数据等操作.这些操作共同驱动 `data plane`的行为。

`control plane` 由四部分组成。

- Controller - `controller deployment` 由多个容器 (`public-api`, `proxy-api`, `destination`,`tap`) 组成，这些容器组合起来提供了 `control plane` 的功能.
- Web - `web deployment` 提供了 Linkerd dashboard.
- Prometheus -  Linkerd 提供的 metrics 由 Prometheus 收集和存储. 但是这个实例被配置为只用来收集和处理 Linkerd 产生的数据。如果想要与现有的Prometheus集成，可以参考这里[https://linkerd.io/2/tasks/exporting-metrics/](https://linkerd.io/2/tasks/exporting-metrics/) .
- Grafana - Linkerd 提供了许多开箱即用的 dashboard。 可以与现有的Grafana组件结合在一起。

### DataPlane

Linkerd `data plane`,由轻量级代理组成，他们作为 `sidecar`容器与服务代码部署在一起(可以理解为每个pod中，除了有一个container运行service code ，还需要有一个 sidecar container 运行 data proxy)。 为了把 现有的服务加入到 Linkerd `service mesh` 中，就需要重新发布所有的pod以便让pod中包含 `data plane proxy`(`linkerd inject`命令就可以完成这个事情)。并且可以通过 CLI 命令  [Adding Your Service](https://linkerd.io/2/tasks/adding-your-service/) 将服务添加到 `data plane`.

这些代理透明地拦截与每个pod之间的通信，并添加诸如检测和加密（TLS）之类的功能，以及根据相关策略允许和拒绝相关请求。

同时，这些代理并不需要手动进行配置，而是通过 `control plane` 来进行配置。

### Proxy

这些超轻的透明代理是由rust编写的，他们被 install 到每个 服务的pod中，成为 `data plane`的一部分。 它接收pod的所有传入流量，并通过`initContainer`拦截所有传出流量，`initContainer`配置`iptables`以正确转发流量。因为它是 `sidecar` 模式，用来拦截服务的所有传入和传出流量，所以不需要更改代码，甚至可以将其添加到正在运行的服务中。

这些代理包含以下特性:

- 对HTTP，HTTP/2和任意TCP协议透明，零配置代理。
- 自动导出HTTP 和 TCP 流量的Prometheus metric指标。
- 透明，零配置WebSocket代理。
- 自动，延迟感知，七层负载
- 针对非HTTP的4层负载均衡
- 支持TLS
- 守护进程，分类诊断API

### CLI

`Linkerd CLI` 运行在 node节点上，用来与 `control plane` 和 `data plane` 进行交互。可以用来查看统计数据，实时调试生产中的数据，和安装升级 `data plane 等`.

### Dashboard

Linkerd Dashboard 可以实时查看服务的运行状态.它可用于查看 特定 指标（成功率，请求/秒和延迟），可视化服务依赖性并了解特定服务路由的运行状况.

![Top Line Metrics (出处https://linkerd.io/images/architecture/stat.png)](images/stat.png)

### Grafana

作为 `control plane` 的一个组件, Grafana 提供了一些开箱即用的 Dashboard。可以用来查看很多信息，例如请求状况，甚至pod内部的一些状态。

![Top Line Metrics](images/grafana-top.png)

![Deployment Detail](images/grafana-deployment.png)

![Pod Detail](images/grafana-pod.png)

![Linkerd Health](images/grafana-health.png)

### Prometheus

Prometheus 是一种云原生的监控解决方案，用来收集和存储所有的Linkerd metrics。它作为 `control plane`的一部分，并且为 CLI,Dashboard,Grafana 提供相应的数据。

`data plane`中的代理暴露了 `4191` 的端口，用来给Prometheus收集数据，每10秒钟刷新一次。 这些指标 可以给其他的Linkerd 组件使用，例如 CLI 和dashboard。

![Metrics Collection (出处 https://linkerd.io/images/architecture/prometheus.svg)](images/prometheus.svg)



## Linkerd Or Istio

[Linkerd or Istio?](https://itnext.io/linkerd-or-istio-2e3ce781fa3a)


## 参考

- [Istio Handbook——Istio 服务网格进阶实战](http://www.servicemesher.com/istio-handbook/)

