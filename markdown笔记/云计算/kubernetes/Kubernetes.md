# Kubernetes

## 概念与术语

### 为什么需要kubernetes

Kubernetes 是一个可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。 Kubernetes 拥有一个庞大且快速增长的生态系统。Kubernetes 的服务、支持和工具广泛可用。

Kubernetes 为你提供了一个可弹性运行分布式系统的框架。

+ **服务发现和负载均衡**

  Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。

+ **存储编排**

  Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。

+ **自动部署和回滚**

  你可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态 更改为期望状态。例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。

+ **自动完成装箱计算**

  Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。 当容器指定了资源请求时，Kubernetes 可以做出更好的决策来管理容器的资源。

+ **自我修复**

  Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的 运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。

+ **密钥与配置管理**

  Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

+ 





### kuberneters 组件

![](Kubernetes.assets/components-of-kubernetes.svg)

#### 节点（Node）组件

节点可以是一个虚拟机或者物理机器，取决于所在的集群配置。Kubernetes 通过将容器放入在节点（Node）上运行的 Pod 中来执行你的工作负载。每个节点包含运行 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 所需的服务， 这些 Pods 由 [控制面板](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane) 负责管理。

##### 节点类型

+ **主节点master**：承载Kuberneters控制管理整个集群的系统的**控制面板**。
+ 工作节点：运行用户实际部署的应用。

##### 工作节点组件

+ [kubelet](https://kubernetes.io/docs/reference/generated/kubelet)

  kubelet 是在每个 Node 节点上运行的 “节点代理”，保证容器运行在Pod中。kubelet 是基于 PodSpec 来工作的。**每个 PodSpec 是一个描述 Pod 的 YAML 或 JSON 对象。** kubelet 接受通过各种机制（主要是通过 apiserver）提供的一组 PodSpec，并确保这些 PodSpec 中描述的容器处于运行状态且运行状况良好。kubelet 不管理不是由 Kubernetes 创建的容器。

  与apiserver 通信、管理所在节点的容器。

+ [容器运行时 container runtime](https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes)

  pod运行在 ‘容器运行时’ 上，如docker

+  [kube-proxy](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-proxy/)

  集群中每个节点上运行的网络代理、负责组件之间的负载均衡网络流量，实现 Kubernetes [服务（Service）](https://kubernetes.io/zh/docs/concepts/services-networking/service/) （服务发现）概念的一部分。**kube-proxy 维护节点上的网络规则**。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。

  网络代理反映了每个节点上 Kubernetes API 中定义的服务，并且可以执行简单的 TCP、UDP 和 SCTP 流转发，或者在一组后端进行循环 TCP、UDP 和 SCTP 转发。

+ 1

##### 工作节点注册

+ 节点上的 `kubelet` 向控制面执行自注册；
+ 手动添加一个 Node 对象。

**Kubernetes 会在内部创建一个 Node 对象作为节点的表示**。Kubernetes 检查 `kubelet` 向 API 服务器注册节点时使用的 `metadata.name` 字段是否匹配。 如果节点是健康的（即所有必要的服务都在运行中），则该节点可以用来运行 Pod。 否则，直到该节点变为健康之前，所有的集群活动都会忽略该节点。

##### 工作节点状态

+ 地址 Addresses

  + HostName：由节点的内核设置。可以通过 kubelet 的 `--hostname-override` 参数覆盖。
  + ExternalIP：通常是节点的可外部路由（从集群外可访问）的 IP 地址。
  + InternalIP：通常是节点的仅可在集群内部路由的 IP 地址。

+ 状况 Conditions ：节点状态

  | `Ready`              | 如节点是健康的并已经准备好接收 Pod 则为 `True`；`False` 表示节点不健康而且不能接收 Pod；`Unknown` 表示节点控制器在最近 `node-monitor-grace-period` 期间（默认 40 秒）没有收到节点的消息 |
  | -------------------- | ------------------------------------------------------------ |
  | `DiskPressure`       | `True` 表示节点的空闲空间不足以用于添加新 Pod, 否则为 `False` |
  | `MemoryPressure`     | `True` 表示节点存在内存压力，即节点内存可用量低，否则为 `False` |
  | `PIDPressure`        | `True` 表示节点存在进程压力，即节点上进程过多；否则为 `False` |
  | `NetworkUnavailable` | `True` 表示节点网络配置不正确；否则为 `False`                |

  

+ 容量与可分配

  描述节点上的可用资源：CPU、内存和可以调度到节点上的 Pod 的个数上限。`capacity` 块中的字段标示节点拥有的资源总量。 `allocatable` 块指示节点上可供普通 Pod 消耗的资源量。

+ 信息

  关于节点的一般性信息，例如内核版本、Kubernetes 版本（`kubelet` 和 `kube-proxy` 版本）、 Docker 版本（如果使用了）和操作系统名称。这些信息由 `kubelet` 从节点上搜集而来。

+ 1

##### 心跳机制

Kubernetes 节点发送的心跳（Heartbeats）有助于确定节点的可用性。 心跳有两种形式：`NodeStatus` 和 [`Lease` 对象](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#lease-v1-coordination-k8s-io)。 每个节点在 `kube-node-lease`[名字空间](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/namespaces/) 中都有一个与之关联的 `Lease` 对象。 `Lease` 是一种轻量级的资源，可在集群规模扩大时提高节点心跳机制的性能。

##### 节点容量

Node 对象会跟踪节点上资源的容量（例如可用内存和 CPU 数量）。 通过[自注册](https://kubernetes.io/zh/docs/concepts/architecture/nodes/#self-registration-of-nodes)机制生成的 Node 对象会在注册期间报告自身容量。 如果你[手动](https://kubernetes.io/zh/docs/concepts/architecture/nodes/#manual-node-administration)添加了 Node，你就需要在添加节点时 手动设置节点容量。

Kubernetes [调度器](https://kubernetes.io/docs/reference/generated/kube-scheduler/)保证节点上 有足够的资源供其上的所有 Pod 使用。它会检查节点上所有容器的请求的总和不会超过节点的容量。 总的请求包括由 kubelet 启动的所有容器，但不包括由容器运行时直接启动的容器， 也不包括不受 `kubelet` 控制的其他进程。



#### 控制面版（Control Plane Components）

控制面板运行于master节点、控制平面的组件对集群做出全局决策(比如调度)，以及检测和响应集群事件（例如，当不满足部署的 `replicas` 字段时，启动新的 [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)）。

注：控制平面组件可以在集群中的任何节点上运行。 然而，为了简单起见，**设置脚本通常会在同一个计算机上启动所有控制平面组件，并且不会在此计算机上运行用户容器。**此节点即为 master节点。

控制面板其中包含如下组件

##### [kube-apiserver](https://kubernetes.io/zh/docs/concepts/overview/components/#kube-apiserver)

API 服务器是 Kubernetes [控制面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane)的组件， 该组件公开了 Kubernetes API。 API 服务器是 Kubernetes 控制面版的通信中心。

api服务器为控制面板的核心， API 服务器负责提供 HTTP API，以供用户、集群中的不同部分和集群外部组件相互通信。Kubernetes API 使你可以查询和操纵 Kubernetes API 中对象（例如：Pod、Namespace、ConfigMap 和 Event）的状态。

##### etcd

etcd 是兼具一致性和高可用性的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库。

##### [kube-scheduler](https://kubernetes.io/zh/docs/concepts/overview/components/#kube-scheduler)

负责监视新创建的、未指定运行[节点（node）](https://kubernetes.io/zh/docs/concepts/architecture/nodes/)的 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) ，选择节点让 Pod 在上面运行。即监听 apiserver 来获取 PodSpec.NodeName 为空的 pod，然后为每个这样的 pod 创建一个 binding 指示 pod 应该调度到哪个节点上。

调度决策考虑的因素包括单个 Pod 和 Pod 集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载间的干扰和最后时限。

##### kube-controller-manager

Master节点上的控制器组合、控制集群级别的功能。

从逻辑上讲，每个[控制器](https://kubernetes.io/zh/docs/concepts/architecture/controller/)都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。控制器如下：

- 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应
- 任务控制器（Job controller）: 监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- 端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)
- 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌

##### cloud-controller-manager

云控制器管理器是指嵌入特定云的控制逻辑的 控制平面组件。 云控制器管理器允许您链接聚合到云提供商的应用编程接口中， 并分离出相互作用的组件与您的集群交互的组件。

`cloud-controller-manager` 仅运行特定于云平台的控制回路。 如果你在自己的环境中运行 Kubernetes，或者在本地计算机中运行学习环境， 所部署的环境中不需要云控制器管理器。



#### 集群插件（Addons）

插件使用 Kubernetes 资源（[DaemonSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/)、 [Deployment](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)等）实现集群功能。 因为这些插件提供集群级别的功能，插件中命名空间域的资源属于 `kube-system` 命名空间。

+ DNS

  几乎所有 Kubernetes 集群都应该 有[集群 DNS](https://kubernetes.io/zh/docs/concepts/services-networking/dns-pod-service/)， 因为很多示例都需要 DNS 服务。集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。kubernetes启动的容器自动添加集群dns服务器包含在dns搜索列表中。

+ Web界面

  [Dashboard](https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/) 是Kubernetes 集群的通用的、基于 Web 的用户界面。 它使用户可以**管理集群**中运行的应用程序以及集群本身并进行故障排除。

+ 容器资源监控

  [容器资源监控](https://kubernetes.io/zh/docs/tasks/debug-application-cluster/resource-usage-monitoring/) 将关于容器的一些常见的时间序列度量值保存到一个集中的数据库中，并提供用于浏览这些数据的界面。

+ 集群层面日志

  [集群层面日志](https://kubernetes.io/zh/docs/concepts/cluster-administration/logging/) 机制负责将容器的日志数据 保存到一个集中的日志存储中，该存储能够提供搜索和浏览接口。



### Kubernetes 对象

在 Kubernetes 系统中，*Kubernetes 对象* 是**持久化的实体**。 Kubernetes 使用这些实体去表示整个集群的状态。特别地，它们描述了如下信息：

- 哪些容器化应用在运行（以及在哪些节点上）
- 可以被应用使用的资源
- 关于应用运行时表现的策略，比如重启策略、升级策略，以及容错策略

Kubernetes 对象是 “目标性记录” —— 一旦创建对象，Kubernetes 系统将持续工作以确保对象存在。 通过创建对象，本质上是在告知 Kubernetes 系统，所需要的集群工作负载看起来是什么样子的， 这就是 Kubernetes 集群的 **期望状态（Desired State）**。

#### 对象规约（Spec）与状态（Status）

几乎每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置： 对象 *`spec`（规约）* 和 对象 *`status`（状态）* 。 对于具有 `spec` 的对象，你必须在创建对象时设置其内容，描述你希望对象所具有的特征： *期望状态（Desired State）* 。

`status` 描述了对象的 *当前状态（Current State）*，它是由 Kubernetes 系统和组件 设置并更新的。在任何时刻，Kubernetes [控制平面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane) 都一直积极地管理着对象的实际状态，以使之与期望状态相匹配。

Kubernetes 中的 Deployment 对象能够表示运行在集群中的应用。 当创建 Deployment 时，设置 Deployment 的 `spec`，指定该应用需要有 3 个副本运行。 Kubernetes 系统读取 Deployment 规约，并启动我们所期望的应用的 3 个实例 —— **更新状态以与规约相匹配**。 如果这些实例中有的失败了（一种状态变更），Kubernetes 系统通过执行修正操作 来响应规约和状态间的不一致 —— 在这里意味着它会启动一个新的实例来替换。

##### Deployment对象

Deployment对象，顾名思义，是用于部署应用的对象。它使Kubernetes中最常用的一个对象，它为ReplicaSet和Pod的创建提供了一种声明式的定义方法，从而无需手动创建ReplicaSet和Pod对象（使用Deployment而不直接创建ReplicaSet是因为Deployment对象拥有许多ReplicaSet没有的特性，例如滚动升级和回滚）。

#### 对象名称和IDS

集群中的每一个对象都有一个[*名称*](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/names/#names) 来标识在同类资源中的唯一性。

每个 Kubernetes 对象也有一个[*UID*](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/names/#uids) 来标识在整个集群中的唯一性。

比如，在同一个[名字空间](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/namespaces/) 中有一个名为 `myapp-1234` 的 Pod, 但是可以命名一个 Pod 和一个 Deployment 同为 `myapp-1234`.

对于用户提供的非唯一性的属性，Kubernetes 提供了 [标签（Labels）](https://kubernetes.io/zh/docs/concepts/working-with-objects/labels)和 [注解（Annotation）](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/annotations/)机制。





#### 描述 Kubernetes 对象

创建 Kubernetes 对象时，必须提供对象的规约，用来描述该对象的期望状态， 以及关于对象的一些基本信息（例如名称）。 当使用 Kubernetes API 创建对象时（或者直接创建，或者基于`kubectl`）， API 请求必须在请求体中包含 JSON 格式的信息。 **大多数情况下，需要在 .yaml 文件中为 `kubectl` 提供这些信息**。 `kubectl` 在发起 API 请求时，将这些信息转换成 JSON 格式。



## 容器

### 私有仓库镜像引入 pod

- 配置节点向私有仓库进行身份验证
  - 所有 Pod 均可读取任何已配置的私有仓库
  - 需要集群管理员配置节点
- 预拉镜像
  - 所有 Pod 都可以使用节点上缓存的所有镜像
  - 需要所有节点的 root 访问权限才能进行设置
- 在 Pod 中设置 ImagePullSecrets
  - 只有提供自己密钥的 Pod 才能访问私有仓库







## Kubernetes Operators

### 基础概念

Kubernetes Operator概念是由CoreOS的工程师于2016年提出的，它是在Kubernetes集群上构建和驱动每个应用程序的高级原生方法，需要特定领域的知识。通过与Kubernetes API的密切合作，它提供了一种一致的方法来自动处理所有应用程序操作流程，而无需任何人工响应。换句话说，Operator是打包，运行和管理Kubernetes应用程序的一种方式。



## Pod

### 玩转pod调度

#### kubernetes 调度器

在 Kubernetes 中，*调度* 是指将 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 放置到合适的 [Node](https://kubernetes.io/zh/docs/concepts/architecture/nodes/) 上，然后对应 Node 上的 [Kubelet](https://kubernetes.io/docs/reference/generated/kubelet) 才能够运行这些 pod。

##### kube-scheduler

###### 过程

一个控制面进程，负责将 Pods 指派到节点上。 调度器基于约束和可用资源为调度队列中每个 Pod 确定其可合法放置的节点。 调度器之后对所有合法的节点进行排序，将 Pod 绑定到一个合适的节点。 在同一个集群中可以使用多个不同的调度器；

+ 过滤

  过滤阶段会将所有满足 Pod 调度需求的 Node 选出来，得出一个 Node 列表，里面包含了所有可调度节点；如果列表为空，代表这个pod不可调度。

+ 打分

  调度器会为 Pod 从所有可调度节点中选取一个最合适的 Node。 根据当前启用的打分规则，调度器会给每一个可调度节点进行打分。最后调度器会把pod 调度到得分最高的一个node上去



###### 调度策略 ？

###### 调度配置 ？



### nodeSelector

约束pod在特定的node上运行

**内置的节点标签**， 需注意可改动

- [`kubernetes.io/hostname`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-hostname) 
- [`failure-domain.beta.kubernetes.io/zone`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#failure-domainbetakubernetesiozone)
- [`failure-domain.beta.kubernetes.io/region`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#failure-domainbetakubernetesioregion)
- [`topology.kubernetes.io/zone`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#topologykubernetesiozone)
- [`topology.kubernetes.io/region`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#topologykubernetesiozone)
- [`beta.kubernetes.io/instance-type`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#beta-kubernetes-io-instance-type)
- [`node.kubernetes.io/instance-type`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#nodekubernetesioinstance-type)
- [`kubernetes.io/os`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-os)
- [`kubernetes.io/arch`](https://kubernetes.io/zh/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-arch)



### 亲和性、反亲和性

#### 节点亲和性

点亲和性概念上类似于 `nodeSelector`，它使你可以根据节点上的标签来约束 Pod 可以调度到哪些节点。

+ requiredDuringSchedulingIgnoredDuringExecution

  硬需求，*必须*满足的规则与nodeSelector一致

  如果你指定了多个与 `nodeAffinity` 类型关联的 `nodeSelectorTerms`，则 **如果其中一个** `nodeSelectorTerms` 满足的话，pod将可以调度到节点上。

  如果你指定了多个与 `nodeSelectorTerms` 关联的 `matchExpressions`，则 **只有当所有** `matchExpressions` 满足的话，Pod 才会可以调度到节点上。

+ preferredDuringSchedulingIgnoredDuringExecution

  调度器将尝试执行但不能保证的*偏好*，如果无法满足，可将pod调度到其他项目中去
  
  weight 范围是 1-100。 对于每个符合所有调度要求（资源请求、RequiredDuringScheduling 亲和性表达式等） 的节点，调度器将遍历该字段的元素来计算总和，并且如果节点匹配对应的 MatchExpressions，则添加“权重”到总和。 然后将这个评分与该节点的其他优先级函数的评分进行组合。 总分最高的节点是最优选的。

注： IgnoredDuringExecution 意味着已调度到node上后，node运行中发生改变，不再满足于pod的亲和性规则，pod可在该node上继续运行。

通过 `spec.affinity`

```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

此节点亲和性规则表示，Pod 只能放置在具有标签键 `kubernetes.io/e2e-az-name` 且标签值为 `e2e-az1` 或 `e2e-az2` 的节点上。 另外，在满足这些标准的节点中，具有标签键为 `another-node-label-key` 且标签值为 `another-node-label-value` 的节点应该优先使用。

同时指定了 `nodeSelector` 和 `nodeAffinity`，*两者*必须都要满足， 才能将 Pod 调度到候选节点上。

##### topologyKey

指定为一个拓扑域，可以使用内置的node标签

od 间亲和性与反亲和性使你可以 *基于已经在节点上运行的 Pod 的标签* 来约束 Pod 可以调度到的节点，而不是基于节点上的标签。 规则的格式为“如果 X 节点上已经运行了一个或多个 满足规则 Y 的 Pod， 则这个 Pod 应该（或者在反亲和性的情况下不应该）运行在 X 节点”。 Y 表示一个具有可选的关联命令空间列表的 LabelSelector； 与节点不同，因为 Pod 是命名空间限定的（因此 Pod 上的标签也是命名空间限定的）， 因此作用于 Pod 标签的标签选择算符必须指定选择算符应用在哪个命名空间。 从概念上讲，**X 是一个拓扑域，如节点、机架、云供应商可用区、云供应商地理区域等。 你可以使用 `topologyKey` 来表示它，`topologyKey` 是节点标签的键以便系统 用来表示这样的拓扑域。**

**topologyKey 的取值范围：**

1. 对于 Pod 亲和性而言，在 `requiredDuringSchedulingIgnoredDuringExecution` 和 `preferredDuringSchedulingIgnoredDuringExecution` 中，`topologyKey` 不允许为空。
2. 对于 Pod 反亲和性而言，`requiredDuringSchedulingIgnoredDuringExecution` 和 `preferredDuringSchedulingIgnoredDuringExecution` 中，`topologyKey` 都不可以为空。
3. 对于 `requiredDuringSchedulingIgnoredDuringExecution` 要求的 Pod 反亲和性， 准入控制器 `LimitPodHardAntiAffinityTopology` 被引入以确保 `topologyKey` 只能是 `kubernetes.io/hostname`。如果你希望 `topologyKey` 也可用于其他定制 拓扑逻辑，你可以更改准入控制器或者禁用之。
4. 除上述情况外，`topologyKey` 可以是任何合法的标签键。



### 污点和容忍度

[*节点亲和性*](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) 是 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 的一种属性，它使 Pod 被吸引到一类特定的[节点](https://kubernetes.io/zh/docs/concepts/architecture/nodes/) （这可能出于一种偏好，也可能是硬性要求）。 *污点*（Taint）则相反——它使节点能够排斥一类特定的 Pod。

容忍度（Toleration）是应用于 Pod 上的，允许（但并不要求）Pod 调度到带有与之匹配的污点的节点上。

污点和容忍度（Toleration）相互配合，可以用来避免 Pod 被分配到不合适的节点上。 每个节点上都可以应用一个或多个污点，这表示对于那些不能容忍这些污点的 Pod，是不会被该节点接受的。



#### 污点

给节点 `node1` 增加一个污点，它的键名是 `key1`，键值是 `value1`，效果是 `NoSchedule`。 这表示只有拥有和这个污点相匹配的容忍度的 Pod 才能够被分配到 `node1` 这个节点。

```
kubectl taint nodes node1 key1=value1:NoSchedule
```

#### 容忍度

给pod设置容忍度，PodSpec 中定义 Pod 的容忍度。 下面两个容忍度均与上面例子中使用 `kubectl taint` 命令创建的污点相匹配， 因此如果一个 Pod 拥有其中的任何一个容忍度都能够被分配到 `node1` ：

```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
  
---
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```

一个容忍度和一个污点相“匹配”是指它们有一样的键名和效果，并且：

- 如果 `operator` 是 `Exists` （此时容忍度不能指定 `value`），或者
- 如果 `operator` 是 `Equal` ，则它们的 `value` 应该相等

如果一个容忍度的 `key` 为空且 operator 为 `Exists`， 表示这个容忍度与任意的 key 、value 和 effect 都匹配，即这个容忍度能容忍任意 taint。

如果 `effect` 为空，则可以与所有键名 `key1` 的效果相匹配。

#### effect 字段

+ NoSchedule
+ PreferNoSchedule
+ NoExecute

一个节点添加多个污点，也可以给一个 Pod 添加多个容忍度设置。 Kubernetes 处理多个污点和容忍度的过程就像一个过滤器：从一个节点的所有污点开始遍历， 过滤掉那些 Pod 中存在与之相匹配的容忍度的污点。余下未被过滤的污点的 effect 值决定了 Pod 是否会被分配到该节点，特别是以下情况：

- 如果未被过滤的污点中存在至少一个 effect 值为 `NoSchedule` 的污点， 则 Kubernetes 不会将 Pod 分配到该节点。
- 如果未被过滤的污点中不存在 effect 值为 `NoSchedule` 的污点， 但是存在 effect 值为 `PreferNoSchedule` 的污点， 则 Kubernetes 会 *尝试* 不将 Pod 分配到该节点。
- 如果未被过滤的污点中存在至少一个 effect 值为 `NoExecute` 的污点， 则 Kubernetes 不会将 Pod 分配到该节点（如果 Pod 还未在节点上运行）， 或者将 Pod 从该节点驱逐（如果 Pod 已经在节点上运行）。

**当一个pod可以调度到一个节点时，必须容忍该节点上的所有污点**

##### 污点驱除

前文提到过污点的 effect 值 `NoExecute`会影响已经在节点上运行的 Pod

- 如果 Pod 不能忍受 effect 值为 `NoExecute` 的污点，那么 Pod 将马上被驱逐
- 如果 Pod 能够忍受 effect 值为 `NoExecute` 的污点，但是在容忍度定义中没有指定 `tolerationSeconds`，则 Pod 还会一直在这个节点上运行。
- 如果 Pod 能够忍受 effect 值为 `NoExecute` 的污点，而且指定了 `tolerationSeconds`， 则 Pod 还能在这个节点上继续运行这个指定的时间长度。





## 存储

### 券

#### emptyDir

当 Pod 分派到某个 Node 上时，`emptyDir` 卷会被创建，node节点上的一目录、pod中的容器共享；并且在 Pod 在该节点上运行期间，卷一直存在。pod可选择挂载或不挂载。

#### hostPath

`hostPath` 卷能将主机节点文件系统上的文件或目录挂载到你的 Pod 中。

- 运行一个需要访问 Docker 内部机制的容器；可使用 `hostPath` 挂载 `/var/lib/docker` 路径。

- 在容器中运行 cAdvisor 时，以 `hostPath` 方式挂载 `/sys`。

- 允许 Pod 指定给定的 `hostPath` 在运行 Pod 之前是否应该存在，是否应该创建以及应该以什么方式存在。

- 具有相同配置（例如基于同一 PodTemplate 创建）的多个 Pod 会由于节点上文件的不同 而在不同节点上有不同的行为。

- 下层主机上创建的文件或目录只能由 root 用户写入。你需要在 [特权容器](https://kubernetes.io/zh/docs/tasks/configure-pod-container/security-context/) 中以 root 身份运行进程，或者修改主机上的文件权限以便容器能够写入 `hostPath` 卷。

  

#### local

`local` 卷所代表的是某个被挂载的本地存储设备，例如磁盘、分区或者目录。

`local` 卷只能用作静态创建的持久卷。尚不支持动态配置。

与 `hostPath` 卷相比，`local` 卷能够以持久和可移植的方式使用，而无需手动将 Pod 调度到节点。系统通过查看 PersistentVolume 的节点亲和性配置，就能了解卷的节点约束。



#### NFS

`nfs` 卷能将 NFS (网络文件系统) 挂载到你的 Pod 中。 不像 `emptyDir` 那样会在删除 Pod 的同时也会被删除，`nfs` 卷的内容在删除 Pod 时会被保存，卷只是被卸载。 这意味着 `nfs` 卷可以被预先填充数据，并且这些数据可以在 Pod 之间共享。





## 网络

### Linux网络

#### IP-CIDR

##### ipv4

IPV4的地址是一个32位的二进制数，由网络ID和主机ID两部分组成，用来在网络中唯一的标识一台计算机。IP地址通常用四组3位的十进制数表示，中间用**.**分割，例如:**192.168.0.1**。

**ip 分类**

+ A : A类地址用IP地址前8位表示网络ID，用IP地址后24位表示主机ID。[0,127].xxx.xxx.xxx.xxx

+ B:  B类地址用IP地址前16位表示网络ID，用IP地址后16位表示主机ID。[128,191].xxx.xxx.xxx

+ C: C类地址用IP地址前24位表示网络ID，用IP地址后8位表示主机ID。[192－223].xxx.xxx.xxx 

+ D: D类地址用来多播使用，没有网络ID和主机ID之分
+ E: E类地址保留实验用，没有网络ID和主机ID之分

**网络ID、主机ID和子网掩码**

当为一台计算机分配IP地址后，该计算机的IP地址哪部份表示网络ID，哪部份表示主机ID，并不由IP地址所属的类来确定，而是由子网掩码确定。子网确定一个IP地址属于哪一个子网。子网掩码的格式是以连续的255后面跟连续的0表示，其中连续的255这部份表示网络ID；连续0部份表示主机ID。比如，子网掩码255.255.0.0和255.255.255.0。

**CIDR(无类域间路由)**

将子网掩码转换为二进制，就会发现网络ID部分全部是1、主机ID部分全部是0。

CIDR（Classless Inter-Domain Routing，无类域间路由选择）它消除了传统的A类、B类和C类地址以及划分子网的概念，因而可以更加有效地分配IPv4的地址空间。它可以将好几个IP网络结合在一起，使用一种无类别的域际路由选择算法，使它们合并成一条路由从而较少路由表中的路由条目减轻Internet路由器的负担。

CIDR技术用子网掩码中连续的1部份表示网络ID，连续的0部份表示主机ID。比如，网络中包含2000台计算机，只需要用11位表示 主机ID，用21位表网络ID，则子网掩码表示为11111111.11111111.11100000.00000000，转换为十进制则为 255.255.224.0。此时，该网络将包含2046台计算机，既不会造成IP地址的浪费，也不会利用路由器连接网络，增加额外的管理维护量。

CIDR表示方法：IP地址/网络ID的位数，比如192.168.23.35/21，其中用21位表示网络ID。





#### 命名空间

支持网络协议多个实例。Linux引入Linux网络命名空间概念，独立的协议栈隔离在命名空间中、其中包含有 路由表、iptables、nat、套接字（必属于一个命名空间、操作也需在命名空间中进行）虚拟网络设备（物理网络设备 只能关联到root命名空间）等。

命令空间是独立的网络协议栈，相互之间隔离无法进行通信，但通过建立Veth对就可以在两个命名空间进行通信。

#### Veth设备对

引入Veth设备对是为了不同的命名空间之间通信，成对出现可看成一对网卡。Veth设备对一端发送数据可直接发送到另一端。

```
#建立Veth设备对
ip link add veth0 type veth peer name veth1

#查看
ip link show

#将一端甩给另一个namespace
ip link set veth1 netns netns1

#设置ip地址才可以通信
ip netns exec netns1 ip addr add 10.1.1.1/24 dev veth1
ip addr add 10.1.1.2/24 dev veth0 
```

#### 网桥

通过网桥把若干个网络接口连接起来，让网络接口之间的报文能够相互转发。网桥是一个二层虚拟网络设备（MAC地址学习），Linux内核通过一个虚拟的网桥设备（Net Device）来进行桥接的。

```
# 新增一个网桥设备
brctl addbr xxxx
 
# 给网桥配置ip地址
ifconfig brxxx 127.1.1.1
```





### 网络模型

基本原则：每个pod都有一个独立的ip地址，并处于一个扁平的网络空间中，相互可以直接连通。

IP-per-Pod模型：一个pod内部的所有container都共享一个网络堆栈（相当于一个网络命名空间、所有容器的IP地址、网络设备、配置都是共享的），Pod内容器可通过localhost连接相互的port。**pod可认作为一台独立的虚拟机或物理机。**

#### **与Docker 动态端口映射不同**

Docker动态端口映射导致访问者看到的IP与PORT与服务提供者实际绑定的IP与PORT不同（通过NAT映射为新IP与PORT），这会引起应用配置的复杂化，服务注册与发现也会出现问题；外部应用也无法通过容器内的私有ip地址与port进行访问。

#### kubernetes网络模型特点

+ 集群中所有容器都可以不用NAT方式直接和其他容器通信

  ![image-20210806103007723](Kubernetes.assets/image-20210806103007723.png)

  同一个pod内容器共享一个网络空间（同一个linux协议栈）、可通过localhost访问

+ 所有节点可以不用NAT方式和所用容器通信

  ![image-20210806103126178](Kubernetes.assets/image-20210806103126178.png)

+ 容器的地址和外部看到的地址是同一个地址

  ![image-20210806103339469](Kubernetes.assets/image-20210806103339469.png)

+ 1

整个Kubernetes 集群中对pod的IP进行分配，不能有冲突（每个node上的docker0 网络段都不一致）

将pod ip和node ip 关联起来、通过CNI网络插件制定路由表



### Network Policy

#### 功能

对Pod间的网络通信进行限制和准入控制，设置方式为将pod的label作为查询条件，设置允许访问或禁止访问的客户端pod列表。

**级别 pod、namespace**。

#### 设置实现

通过 kubernetes **资源对象 NetworkPolicy** 与 **策略控制器（Policy Controller）**进行策略实现。策略控制器可通过第三方插件提供如（Calico）。

集群使用CNI - Calico

#### 实验案例

**设置namespace下指定 pod 的访问策略Ingress**

+ 同namespace 下的指定pod可访问

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: f5-test-nginx-port2-netpolicy
  namespace: test-czw
spec:
  podSelector:
    matchLabels:
      app-netpolicy: f5-test-nginx-port2-pod #选中执行策略的pod
  policyTypes:
  - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app-netpolicy: f5-test-tomcat-port1-pod  #同namespace 下的指定pod可访问
```

+ 同namespace 下的指定pod可访问 访问指定端口 8080

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: f5-test-nginx-port2-netpolicy
  namespace: test-czw
spec:
  podSelector:
    matchLabels:
      app-netpolicy: f5-test-nginx-port2-pod
  policyTypes:
  - Ingress
  ingress:
    - ports:
        - port: 8080 #指定端口
      from: # 注意在 一个 ports 和 from 在一个 - 下
        - podSelector:
            matchLabels:
              app-netpolicy: f5-test-tomcat-port1-pod
              
---  
# 或，ports 和 from 在一个 - 下
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: f5-test-nginx-port2-netpolicy
  namespace: test-czw
spec:
  podSelector:
    matchLabels:
      app-netpolicy: f5-test-nginx-port2-pod
  policyTypes:
  - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app-netpolicy: f5-test-tomcat-port1-pod
      ports:
        - port: 8080
          protocol: TCP
```



+ 指定其他的namespace访问指定pod 

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: f5-test-nginx-port2-netpolicy
  namespace: test-czw
spec:
  podSelector:
    matchLabels:
      app-netpolicy: f5-test-nginx-port2-pod
  policyTypes:
  - Ingress
  ingress:
    - from:
        - podSelector:  # 同一namespace 下的其他pod
            matchLabels:
              app-netpolicy: f5-test-tomcat-port1-pod
        - namespaceSelector: # 有指定标签的namespace 下的所有pod可访问
            matchLabels:
              app-netpolicy: f5-test-nginx-port2-pod
      ports:
        - port: 8080
          protocol: TCP
          
---
# 指定namespace 下的 指定pod
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: f5-test-nginx-port2-netpolicy
  namespace: test-czw
spec:
  podSelector:
    matchLabels:
      app-netpolicy: f5-test-nginx-port2-pod
  policyTypes:
  - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app-netpolicy: f5-test-tomcat-port1-pod
        - namespaceSelector:
            matchLabels:
              app-netpolicy: f5-test-nginx-port2-pod
          podSelector:
            matchLabels:
              app-netpolicy: f5-test-nginx-port2-pod
      ports:
        - port: 8080
          protocol: TCP


```

+ 命名空间自身隔离、与其他命名空间关联

```
# 本命名空间隔离
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: test-czw-myself
  namespace: test-czw
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              app-netpolicy: test-czw


---
# 命名空间间关联
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: test-czw-and-othhers
  namespace: test-czw
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector:
            matchExpressions:
              - key: app-netpolicy
                operator: In
                values:
                  - test-czw
                  - f5-test-nginx-port2-pod
```

+ 111



## 服务发现 - Service

将运行在一组 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 上的应用程序公开为网络服务的抽象方法。

使用 Kubernetes，你无需修改应用程序即可使用不熟悉的服务发现机制。 Kubernetes 为 Pods 提供自己的 IP 地址，并为一组 Pod 提供相同的 DNS 名， 并且可以在它们之间进行负载均衡。



### 基本组成

| 属性                 | 取值类型 | 是否必填 | 取值说明                                                     |
| -------------------- | -------- | -------- | ------------------------------------------------------------ |
| spec.type            | string   | required | Service类型，即Service的访问方式，默认值为 ClusterIP;   <br>(1) ClusterIP: 虚拟的服务IP地址，用于集群内部pod访问（iptables规则修改）<br>(2)NodePort: 使用宿主机端口，外部通过node ip + 发布端口，访问服务<br>(3)LoadBalancer:使用外部负载均衡器完成到服务的分发，需在spec.status.loadBalancer指定外部负载均衡器地址 |
| spec.clusterIP       | string   |          | 虚拟服务ip地址，无法直接ping 通，和pod ip 不一样，pod有虚拟网卡 |
| spec.sessionAffinity | String   |          | 是否支持session轮询<br>ClientIP : 同一个客户端的访问请求都转发到同一个后端pod |
|                      |          |          |                                                              |



### 无selector的Service

**当使用外部服务时**，可创建一个无selector的Svc，需在同一个namespace下建立一个同名的Endpoint

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
      
---
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42 #外部的ip
    ports:
      - port: 9376 #外部端口

      
      
```



## 配置

### ConfigMap

使用congfimap 将配置文件外置到镜像外部，即配置分离。

ConfigMap 使用 `data` 和 `binaryData` 字段。这些字段能够接收键-值对作为其取值。`data` 和 `binaryData` 字段都是可选的。`data` 字段设计用来保存 UTF-8 字节序列，而 `binaryData` 则 被设计用来保存二进制数据作为 base64 编码的字串。

**在 Pod 中将 ConfigMap 当做文件使用**

1. 创建一个 ConfigMap 对象或者使用现有的 ConfigMap 对象。多个 Pod 可以引用同一个 ConfigMap。
2. 修改 Pod 定义，在 `spec.volumes[]` 下添加一个卷。 为该卷设置任意名称，之后将 `spec.volumes[].configMap.name` 字段设置为对 你的 ConfigMap 对象的引用。
3. 为每个需要该 ConfigMap 的容器添加一个 `.spec.containers[].volumeMounts[]`。 设置 `.spec.containers[].volumeMounts[].readOnly=true` 并将 `.spec.containers[].volumeMounts[].mountPath` 设置为一个未使用的目录名， ConfigMap 的内容将出现在该目录中。
4. 更改你的镜像或者命令行，以便程序能够从该目录中查找文件。ConfigMap 中的每个 `data` 键会变成 `mountPath` 下面的一个文件名。





### Secret

#### 概述

`Secret` 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。Kubernetes Secret 默认情况下存储为 base64-编码的、非加密的字符串。可采取如下策略：

+ 为 Secret [启用静态加密](https://kubernetes.io/zh/docs/tasks/administer-cluster/encrypt-data/)；
+ [启用 或配置 RBAC 规则](https://kubernetes.io/zh/docs/reference/access-authn-authz/authorization/)来限制对 Secret 的读写操作。 要注意，任何被允许创建 Pod 的人都默认地具有读取 Secret 的权限。

Kubernetes 提供若干种内置的类型，用于一些常见的使用场景。 针对这些类型，Kubernetes 所执行的合法性检查操作以及对其所实施的限制各不相同。

| 内置类型                              | 用法                                     |
| ------------------------------------- | ---------------------------------------- |
| `Opaque`                              | 用户定义的任意数据                       |
| `kubernetes.io/service-account-token` | 服务账号令牌                             |
| `kubernetes.io/dockercfg`             | `~/.dockercfg` 文件的序列化形式          |
| `kubernetes.io/dockerconfigjson`      | `~/.docker/config.json` 文件的序列化形式 |
| `kubernetes.io/basic-auth`            | 用于基本身份认证的凭据                   |
| `kubernetes.io/ssh-auth`              | 用于 SSH 身份认证的凭据                  |
| `kubernetes.io/tls`                   | 用于 TLS 客户端或者服务器端的数据        |
| `bootstrap.kubernetes.io/token`       | 启动引导令牌数据                         |

通过为 Secret 对象的 `type` 字段设置一个非空的字符串值，你也可以定义并使用自己 Secret 类型。如果 `type` 值为空字符串，则被视为 `Opaque` 类型。 Kubernetes 并不对类型的名称作任何限制。不过，如果你要使用内置类型之一， 则你必须满足为该类型所定义的所有要求。

#### 使用Secret

Secret 可以作为数据卷被挂载，或作为[环境变量](https://kubernetes.io/zh/docs/concepts/containers/container-environment/) 暴露出来以供 Pod 中的容器使用。它们也可以被系统的其他部分使用，而不直接暴露在 Pod 内。 例如，它们可以保存凭据，系统的其他部分将用它来代表你与外部系统进行交互。

+ Pod中使用Secret文件

```
# 将Secret中每一个键值映射成指定目录下的独立文件
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

+ 将 Secret 键名映射到特定路径

  如果使用了 `spec.volumes[].secret.items`，只有在 `items` 中指定的键会被映射。 要使用 Secret 中所有键，就必须将它们都列在 `items` 字段中。 所有列出的键名必须存在于相应的 Secret 中。否则，不会创建卷。

```
# username Secret 存储在 /etc/foo/my-group/my-username 文件中而不是 /etc/foo/username 中。

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
        mode: 511 # linux 文件权限777
```

**已卷挂载形式的Secret会自动更新**，当已经存储于卷中被使用的 Secret 被更新时，被映射的键也将终将被更新。 组件 kubelet 在周期性同步时检查被挂载的 Secret 是不是最新的。 但是，它会使用其本地缓存的数值作为 Secret 的当前值。



**以环境变量的形式使用 Secrets**

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```

如果某个容器已经在通过环境变量使用某 Secret，对该 Secret 的更新不会被 容器马上看见，除非容器被重启。

imagePullSecret

```
# 拉取镜像Secret
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2015-08-07T22:02:39Z
  name: default
  namespace: default
  uid: 052fb0f4-3d50-11e5-b066-42010af0d7b6
secrets:
- name: default-token-uudge
imagePullSecrets:
- name: myregistrykey
```



## Kubectl 命令

+ 给pod 添加label

  ```
  #添加
  kubectl label pod f5-test-nginx-port2-cf5fc5989-lrqkc app-netpolicy=f5-test-nginx-port2-pod -n test-czw
  #修改
  kubectl label pod f5-test-nginx-port2-cf5fc5989-lrqkc app-netpolicy=f5-test-nginx-port2-pod-update -n test-czw -- overwrite
  #删除
  kubectl label pod f5-test-nginx-port2-cf5fc5989-lrqkc app-netpolicy- -n test-czw
  ```

  

+ 进入pod

  ```
  kubectl exec -it f5-test-tomcat-port1-568c67d9fb-f5sr9 -n test-czw -- /bin/bash
  ```

  

+ 批量删除pod

  ```
  #筛选 命名空间下 状态 Completed
  kubectl get pods -n databench-ns | grep Completed | awk '{print $1}'
  
  #批量删除
  kubectl get pods -n databench-ns | grep Completed | awk '{print $1}' | xargs kubectl delete pod
  
  #强制批量删除
  kubectl get pods -n cce-system | grep Terminating | grep bobft-cce-web-server-nginx | awk '{print $1}' | xargs kubectl delete pod --grace-period=0 --force
  
  kubectl get pods -n cce-system | grep bobft-cce-web-server-nginx-68f8bbfd64 | awk '{print $1}' | xargs kubectl delete pod -n cce-system --force --grace-period=0
  
  kubectl get pods -n tekton-pipelines | grep Termina | awk '{print $1}' | xargs kubectl delete pods -n tekton-pipelines --grace-period=0 --force
  ```

+ 强制删除pod

  ```
  kubectl delete pod bobft-cce-web-server-nginx-68f8bbfd64-zzv7m -n cce-system --grace-period=0 --force
  ```

  

+ 查看异常pod数量

  ```
  kubectl get pods -n tekton-pipelines | grep Terminating | grep tekton-pipelines-webhook | wc -l 
  ```

  

+ 查看 notReady node的 kubelet

  ```
  systemctl status kubelet
  systemctl restart kubelet
  ```

  

+ 查询etcd

```
systemctl status etcd -l
```

+ 查询kube-apiserver

  ```
  systemctl status kube-apiserver
  ```

+ 查看node

  ```
  kubectl get node
  ```

  

+ 查看pod日志

+ 

```
kubectl logs -f bobft-cce-api-server-6d66d9464-bnrcj -c bobft-cce-api-server -n cce-system
```

+ 直接操作Etcd

  ```
  export ETCDCTL_API=3 && etcdctl --cacert=/etc/kubernetes/ssl/ca.pem  --cert=/etc/kubernetes/ssl/kubernetes.pem  --key=/etc/kubernetes/ssl/kubernetes-key.pem  --endpoints=https://172.16.5.43:2379 get /cn/com/bobfintech/cce/dockerRegistrySecret
  ```

  ```
  export ETCDCTL_API=3 && etcdctl --cacert=/etc/kubernetes/ssl/ca.pem  --cert=/etc/kubernetes/ssl/kubernetes.pem  --key=/etc/kubernetes/ssl/kubernetes-key.pem  --endpoints=https://172.16.5.43:2379 get / --prefix --keys-only|grep cce-system|grep bobft-cce-web-server-nginx 
  ```

  

+ 查看linux状态

```
查看cpu: top 
        lscpu
查看内存： free -h
磁盘： df -h

```



+ 11
+ 11
+ 11
+ 11









