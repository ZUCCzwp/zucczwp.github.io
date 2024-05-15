# 微服务架构


## 单体架构的困难和挑战
### 编译、部署困难
### 代码分支管理困难
因为单体应用非常庞大，所以代码模块也是由多个团队共同维护的。但最后还是要编译成一
个单体应用，统一发布。这就要求把各个团队的代码 merge 在一起，这个过程很容易发生
代码冲突。而 merge 的时候又是应用要发布的时候，发布过程本来就复杂，再加上代码
merge 带来的问题，各种情况纠缠在一起，极易出错。
### 数据库连接耗尽
对于一个巨型的应用而言，因为有大量的用户进行访问，所以必须把应用部署到大规模的服
务器集群上。然后每个应用都需要与数据库建立连接，大量的应用服务器连接到数据库，会
对数据库的连接产生巨大的压力，某些情况下甚至会耗尽数据库的连接。
### 新增业务困难
因为所有的业务都耦合在一个单一的大系统里，随着时间的发展，这个系统会变得非常的复
杂，想要维护这样一个系统是非常困难和复杂的。
### 发布困难
## 微服务框架原理
在面向服务的体系架构里面，服务提供者向注册中心注册自己的服务，而服务调用者到注册
中心去发现服务，发现服务以后，根据服务注册中心提供的访问接口和访问路径对服务发起
请求，由服务的提供者完成请求，返回结果给调用者。
## 微服务架构的落地实践
首先明确自己的需求：我们到底想用微服务达到什么样的目的？需求清晰了，再去考虑具体
要实现的价值，再根据价值构建我们的设计原则，根据原则寻找最佳实践，最后根据实践去
选择最合适的工具。
## 微服务间如何通讯
### 从通讯协议角度考虑
* Rest Api
* RPC
* MQ
### 从通讯模式角度考虑
|    | 一对一        | 一对多         |
|----|------------|-------------|
| 同步 | 请求响应模式，最常见 | --          |
| 异步 | 通知/请求异步响应  | 发布订阅/发布异步响应 |
### 如何选择 RPC框架
* I/O、线程调度模型
* 序列化方式
* 多语言支持
## 服务发现
### 客户端发现
![client_find.png](/images/client_find.png)
### 服务端发现
![service_find.png](/images/service_find.png)

## 服务编排
### kubernetes
![k8s_cluster.png](/images/k8s_cluster.png)

六边形表示的是 node节点，即 worker节点。每个工作节点上运行了kubelet服务以及 docker服务。
kubelet就相当于是该节点运行时的一个总管，他会管理这个当前 node上面运行的所有服务

deployment相当于是一个部署，圆圈相当于是一个 pod.pod是k8s概念中的最小单元
![pod.png](/images/pod.png)

pod里边是容器。有独立的 ip地址。pod里面的容器可以共享网络以及存储的。

![k8s_service.png](/images/k8s_service.png)

虚线表示 service,service也有自己独立的 ip。可对多个 pod进行负载均衡。使用 label selector来定义。

![k8s_architecture.png](/images/k8s_architecture.png)

master虚线框代表的是 api server,他提供了资源操作的唯一入口，并且提供了认证、授权和访问控制。control manager负责维护集群的状态，
scheduler负责资源调度。按照预定的调度策略。把 pod调度给相应的 node节点上，etcd主要用于一致性存储，保存了集群状态等等的信息。

kubelet负责维护当前节点上的容器的生命周期，也负责维护当前的节点的 volume和网络的管理。 kube-proxy负责 service提供内部的服务发现和负载均衡
kube-DNS负责整个集群的 DNS服务

dashboard提供一些集群相关的数据的展示和操作。

集群外要访问集群内部服务时，k8s可以把服务端口直接暴露在当前的 node上。直接访问 node的 ip就可以关联到这个端口

#### 设计理念
* API设计原则（申明式）
* 控制机设计原则

#### 网络
* CNI
* Flannel、Calico、Weave
* Pod网络
 
#### scheduler-preselect(预选规则)
* NodiskConflict(挂载冲突)
* CheckNodeMemoryPressure
* NodeSelector
* FitResource
* Affinity

#### scheduler-optimize-select(优选规则)
SelectorSpreadPriority
LeastRequestedPriority
AffinityPriority

#### pod通讯
* 同一个 pod内部通讯 用localhost地址访问彼此的端口。
* 同一个 node,不同 pod通讯 可通过 docker0网桥进行通讯，具体可通过访问pod的ip
* 不同 node，不同 pod通讯 通过 pod的 ip和 node的 ip关联

#### 服务发现
* kube-proxy(ClusterIp)
* kube-proxy(NodePort)
* kube-DNS

## CICD和 DevOps
持续集成
持续部署
