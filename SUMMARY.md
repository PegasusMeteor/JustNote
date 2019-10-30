# Summary

* [简介](readme.md)

* [DesignPattern](designpattern/readme.md)
  * [七大原则](designpattern/seven-principle.md)
    * [开闭原则(OCP)](designpattern/seven-principle/open-closed-principle.md)
    * [依赖倒置原则(DIP)](designpattern/seven-principle/dependence-inversion-principle.md)
    * [单一职责原则(SRP)](designpattern/seven-principle/single-responsibility-principle.md)
    * [接口隔离原则(ISP)](designpattern/seven-principle/interface-segregation-principle.md)
    * [迪米特法则(LoD)](designpattern/seven-principle/law-of-demeter.md)
    * [里氏代换原则(LSP)](designpattern/seven-principle/liskov-substitution-principle.md)
    * [合成复用原则(CRP)](designpattern/seven-principle/composite-reuse-principle.md)
  * [创建型模式](designpattern/creational-pattern.md)
    * [简单工厂模式](designpattern/creational-principle/simple-factory-pattern.md)
    * [工厂方法模式](designpattern/creational-principle/factory-method-pattern.md)
    * [抽象工厂模式](designpattern/creational-principle/abstract-factory-pattern.md)
    * [建造者模式](designpattern/creational-principle/builder-pattern.md)
    * [单例模式](designpattern/creational-principle/singleton-pattern.md)
    * [原型模式](designpattern/creational-principle/prototype-pattern.md)
  * [结构型模式](designpattern/structural-pattern.md)
    * [外观模式](designpattern/structural-principle/facade-pattern.md)
    * [装饰模式](designpattern/structural-principle/decorator-pattern.md)
    * [适配器模式](designpattern/structural-principle/adapter-pattern.md)
    * [享元模式](designpattern/structural-principle/flyweight-pattern.md)
    * [组合模式](designpattern/structural-principle/composite-pattern.md)
    * [桥接模式](designpattern/structural-principle/bridge-pattern.md)
    * [代理模式](designpattern/structural-principle/proxy-pattern.md)
  * [行为型模式](designpattern/behavioral-pattern.md)
    * [模板方法模式](designpattern/behavioral-principle/template-method-pattern.md)
    * [迭代器模式](designpattern/behavioral-principle/iterator-pattern.md)
    * [策略模式](designpattern/behavioral-principle/strategy-pattern.md)
    * [解释器模式](designpattern/behavioral-principle/interpreter-pattern.md)
    * [观察者模式](designpattern/behavioral-principle/observer-pattern.md)
    * [备忘录模式](designpattern/behavioral-principle/memento-pattern.md)
    * [命令模式](designpattern/behavioral-principle/command-pattern.md)
    * [中介者模式](designpattern/behavioral-principle/mediator-pattern.md)
    * [责任链模式](designpattern/behavioral-principle/chain-of-responsibility-pattern.md)
    * [访问者模式](designpattern/behavioral-principle/visitor-pattern.md)
    * [状态模式](designpattern/behavioral-principle/state-pattern.md)

<!-- * [Vue](vue/readme.md) -->
* [Java](java/readme.md)
  * [Java Core](java/core.md)
    * [JVM 如何加载类](java/core/jvm-load-class.md)
    * [JVM 垃圾回收](java/core/jvm-gc.md)
    * [JVM G1GC](java/core/jvm-g1gc.md)
    * [JVM G1GC Q&A](java/core/jvm-g1gc-qa.md)
    * [JVM 与 Hbase](java/core/jvm-hbase.md)
  <!-- * [Spring](java/spring/readme.md) -->
  <!-- * [Spring boot](java/springboot/readme.md) -->
  <!-- * [Spring cloud](java/springcloud/readme.md) -->
<!-- * [mysql](mysql/readme.md) -->

* [Golang](golang/readme.md)
  * [源码阅读](golang/source.md)
    * [Goroutines](golang/source/goroutine.md)
    * [Channel](golang/source/channel.md)
  <!-- * [GO WEB 编程](golang/gin/readme.md) -->
    <!-- * [快速开始](golang/gin/quick-start.md) -->
  <!-- * [ORM](golang/orm/readme.md) -->
  * [gRPC](golang/grpc.md)
    * [1、快速开始](golang/grpc/quick-start.md)
    * [2、什么是gRPC](golang/grpc/what-grpc.md)
    * [3、gRPC概念梳理](golang/grpc/grpc-concepts.md)
    * [4、基于Golang的gRPC入门](golang/grpc/grpc-basic.md)
    * [5、gRPC组件ProtocolBuffers介绍](golang/grpc/protocol-buffers.md)
    * [6、gRPC组件Http 2.0](golang/grpc/http2.md)
    * [7、错误处理和Debug](golang/grpc/error-debug.md)
    * [8、gRPC身份验证](golang/grpc/authentication.md)
    * [9、服务注册与发现](golang/grpc/consul.md)
    * [10、gRPC与gRPC Gateway](golang/grpc/grpc-gateway.md)
    * [11、gRPC与分布式链路追踪](golang/grpc/grpc-tracing.md)
  <!-- * [gRPC-Web] -->
  <!-- * [函数式编程](golang/functional-programming/readme.md) -->
  <!-- * [RESTful API](golang/restful/readme.md) -->
  <!-- * [爬虫](golang/crawler/readme.md) -->

* [Scala](scala/readme.md)
  * [函数式编程](scala/function-programming/readme.md)
    * [高阶函数](scala/function-programming/highorderfunc.md)
    * [偏函数](scala/function-programming/partialfunc.md)
  * [集合操作](scala/collection-operation.md)
    * [map](scala/collection-operation/map.md)
    * [flatmap](scala/collection-operation/flatmap.md)
    * [filter](scala/collection-operation/filter.md)
    * [reduceLeft](scala/collection-operation/reduceleft.md)
    * [foldLeft](scala/collection-operation/foldleft.md)
  * [Future](scala/future/readme.md)
  * [Akka](scala/akka/readme.md)

<!-- * [Algorithm](algorithm/readme.md)
  * [Array](algorithm/array.md)
    * [两数之和](algorithm/array/two-sum.md)
  * [LinkedList](algorithm/linkedlist.md)
    * [两数相加](algorithm/linkedlist/add-two-numbers.md)
  * [dynamic-programming](algorithm/programming.md)
    * [最长回文子串](algorithm/programming/longest-palindromic-substring.md) -->

* [Docker](docker/readme.md)

* [Kubernetes](kubernetes/k8s.md)
  * [二进制安装kubernetes](kubernetes/k8s/binary.md)
    * [00.从零开始](kubernetes/k8s/binary/start-zero.md)

* [Architecture](architecture/readme.md)
  * [Infrastructure](architecture/infrastructure.md)
    * [Opentracing](architecture/infrastructure/opentracing.md)
    * [Jaeger && ZipKin](architecture/infrastructure/jaeger-zipkin.md)
    * [Consul](architecture/infrastructure/consul.md)
    * [Envoy](architecture/infrastructure/envoy.md)
    * [Service Mesh](architecture/infrastructure/service-mesh.md)
    * [Service Mesh: Istio 详解](architecture/infrastructure/service-mesh-istio.md)
    * [Service Mesh: 基于 Istio 的落地实践(一)](architecture/infrastructure/service-mesh-istio-practice.md)
