# SkyWalking

SkyWalking 是一个开源可观察性平台，用于收集、分析、聚合和可视化来自服务和云原生基础设施的数据。SkyWalking 提供了一种简单的方法来维护分布式系统的清晰视图，即使是跨云也是如此。它是一种现代 APM，专为云原生、基于容器的分布式系统而设计。

SkyWalking是一个开源的 APM 系统，包括对 Cloud Native 架构中分布式系统的监控、跟踪、诊断能力。核心功能如下。

- 服务、服务实例、端点指标分析
- 根本原因分析。在运行时分析代码
- 服务拓扑图分析
- 服务、服务实例和端点依赖分析
- 缓慢的服务和端点检测
- 性能优化
- 分布式跟踪和上下文传播
- 数据库访问指标。检测慢速数据库访问语句（包括SQL语句）
- 消息队列性能和消耗延迟监控
- 警报
- 浏览器性能监控
- 基础设施（VM、网络、磁盘等）监控
- 跨指标、跟踪和日志的协作

详细的定义解释  [SkyWalking Docs](https://skywalking.apache.org/docs/main/v8.8.1/en/concepts-and-designs/overview/)
 

## SkyWalking 架构

![SkyWalking Architecture](images/SkyWalking_Architecture_20210424.png)

整个架构，分成上、下、左、右四部分：

        考虑到让描述更简单，舍弃掉 Metric 指标相关，而着重在 Tracing 链路相关功能。

- 上部分 **Agent** ：负责从应用中，收集链路信息，发送给 SkyWalking OAP 服务器。目前支持 SkyWalking、Zikpin、Jaeger 等提供的 Tracing 数据信息。而我- 们目前采用的是，SkyWalking Agent 收集 SkyWalking Tracing 数据，传递给服务器。
- 下部分 **SkyWalking OAP** ：负责接收 Agent 发送的 Tracing 数据信息，然后进行分析(Analysis Core) ，存储到外部存储器( Storage )，最终提供查询( - Query )功能。
- 右部分 **Storage** ：Tracing 数据存储。目前支持 ES、MySQL、Sharding Sphere、TiDB、H2 多种存储器。而我们目前采用的是 ES ，主要考虑是 SkyWalking - 开发团队自己的生产环境采用 ES 为主。
- 左部分 **SkyWalking UI** ：负责提供控台，查看链路等等。


## 编程语言或者数据来源支持

1. Java, .NET Core, NodeJS, PHP, and Python auto-instrument agents.
2. Go and C++ SDKs.
3. LUA agent especially for Nginx, OpenResty and Apache APISIX.
4. Browser agent.
5. Service Mesh Observability. Control panel and data panel.
6. Metrics system, including Prometheus, OpenTelemetry, Spring Sleuth(Micrometer), Zabbix.
7. Logs.
8. Zipkin v1/v2 trace.(No Analysis)

## 开源社区

_ |Jaeger
-|-
**Contributors** | 382 
**Open Issues** | 56 
**Open PRs**  | 3 
**GitHub Stars** | 18k 

## 参考资料
- [SkyWalking Docs](https://skywalking.apache.org/docs/#SkyWalking)

