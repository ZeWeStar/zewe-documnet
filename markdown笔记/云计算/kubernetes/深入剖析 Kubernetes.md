# 深入剖析 Kubernetes

## 基础概念

### 知识点

+ 金丝雀部署：

  优先发布一台或少量机器升级，等验证无误后再更新其他机器。优点是用户影响范围小，不足之处是要额外控制如何做自动更新。

+ 蓝绿部署：

  2组机器，蓝代表当前的V1版本，绿代表已经升级完成的V2版本。通过LB将流量全部导入V2完成升级部署。优点是切换快速，缺点是影响全部用户。

+ 

### 云类型

+ 私有云

  私有云是专为单个组织运营的云基础架构，管理的模式有内部管理，第三方管理，亦或是内部或外部托管。私有云就是通过自建或者租用场地的形式建立服务器机房或者数据中心。服务是面向私有网络或者VPN专有网络。企业拥有对服务器、[数据硬盘](https://www.zhihu.com/search?q=数据硬盘&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743669668})的完全控制。因此安全性很高。

+ 公有云

  公有云服务面向公开网络暴露，服务可能也是免费的。由于网络对外公布，因此从安全层面上也是大不相同的。常见的公有云有AWS，Microsoft Azure，阿里云等

+ 混合云

  混合云是两个或多个云（私有云，社区云或公共云）的组合，它们保持不同的实体但绑定在一起，提供多个部署模型的好处。 混合云还意味着能够使用云资源连接搭配，托管和/或专用服务。

  

### 云服务提供三种类型

- **IaaS**：基础设施服务，Infrastructure-as-a-service

  用户可以在云服务提供商提供的基础设施上部署和运行任何软件，包括[操作系统](https://www.zhihu.com/search?q=操作系统&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743669668})和应用软件。用户没有权限管理和访问底层的基础设施，如**服务器、交换机、硬盘**等（不需要自己准备），但是有权管理操作系统、存储内容，可以安装管理应用程序，甚至是有权管理[网络组件](https://www.zhihu.com/search?q=网络组件&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743669668})。简单的说用户使用IaaS，有权管理操作系统之上的一切功能。我们常见的IaaS服务有虚拟机、虚拟网络、以及存储。

  

- **PaaS**：平台服务，Platform-as-a-service

  PaaS给用户提供的能力是使用由云服务提供商支持的编程语言、库、服务以及开发工具来创建、开发[应用程序](https://www.zhihu.com/search?q=应用程序&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743669668})并部署在相关的基础设施上。用户无需管理底层的基础设施，包括网络、服务器，操作系统或者存储。他们只能控制部署在基础设施中操作系统上的应用程序，配置应用程序所托管的环境的可配置参数。常见的PaaS服务有数据库服务、web应用以及容器服务。成熟的PaaS服务会简化开发人员，提供完备的PC端和移动端软件开发套件（SDK），拥有丰富的开发环境（Inteli、Eclipse、VS等），完全可托管的数据库服务，可配置式的应用程序构建，支持多语言的开发，**面向应用市场。**

  直接提供应用服务。



- **SaaS**：软件服务，Software-as-a-service

  SaaS给用户提供的能力是使用在云基础架构上运行的云服务提供商的应用程序。可以通过轻量的[客户端接口](https://www.zhihu.com/search?q=客户端接口&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743669668})（诸如web浏览器（例如，基于web的电子邮件））或程序接口从各种客户端设备访问应用程序。 用户无需管理或控制底层云基础架构，包括网络，服务器，操作系统，存储甚至单独的应用程序功能，可能的例外是有限的用户特定应用程序配置设置。类似的服务有：各类的网盘(Dropbox、[百度网盘](https://www.zhihu.com/search?q=百度网盘&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743669668})等)，JIRA，GitLab等服务。而这些应用的提供者不仅仅是云服务提供商，还有众多的第三方提供商（ISV: independent software provider）。

  如网盘服务、代码托管 gitlab、邮件服务 mail



## 容器

### 概念

+ 容器

  容器其实是一种沙盒技术。顾名思义，沙盒就是能够像一个集装箱一样，把你的应用“装”起来的技术。这样，应用与应用之间，就因为有了边界而不至于相互干扰；而被装进集装箱的应用，也可以被方便地搬来搬去。

+ 进程

  当编写的代码程序通过编译为二进制文件放在计算机中，一旦“程序”被执行起来，它就从磁盘上的二进制文件，变成了计算机内存中的数据、寄存器里的值、堆栈中的指令、被打开的文件，以及各种设备的状态信息的一个集合。像这样一个程序运行起来后的计算机执行环境的总和，就是我们今天的主角：进程。

+ 容器技术

  对于进程来说，它的静态表现就是程序，平常都安安静静地待在磁盘上；而一旦运行起来，它就变成了计算机里的数据和状态的总和，这就是它的动态表现。

  容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个“边界”。

  Docker 等大多数 Linux 容器来说，**Cgroups 技术是用来制造约束的主要手段，而 Namespace 技术则是用来修改进程视图的主要方法。**

  **容器为一种特殊的进程。**

+  Docker 项目

  它最核心的原理实际上就是为待创建的用户进程

  启用 Linux Namespace 配置；

  设置指定的 Cgroups 参数；

  切换进程的根目录（Change Root）。



### 隔离与限制

+ **Namespace**

Namespace 技术实际上修改了应用进程看待整个计算机“视图”，即它的“视线”被操作系统做了限制，只能“看到”某些指定的内容。但对于宿主机来说，这些被“隔离”了的进程跟其他进程并没有太大区别。

**PID Namespace** ：

其实就是对被隔离应用的进程空间做了手脚，使得这些进程只能看到重新计算过的进程编号，比如 PID=1。可实际上，他们在宿主机的操作系统里，还是原来的第 100 号进程。

用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数;

```
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
```

新创建的这个进程将会“看到”一个全新的进程空间，在这个进程空间里，它的 PID 是 1。之所以说“看到”，是因为这只是一个“障眼法”，在宿主机真实的进程空间里，这个进程的 PID 还是真实的数值，比如 100。

Linux 操作系统还提供了 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行“障眼法”操作。比如，Mount Namespace，用于让被隔离进程只看到当前 Namespace 里的挂载点信息；Network Namespace，用于让被隔离进程看到当前 Namespace 里的网络设备和配置。

在创建容器进程时，**指定了这个进程所需要启用的一组 Namespace 参数。这样，容器就只能“看”到当前 Namespace 所限定的资源、文件、设备、状态，或者配置。而对于宿主机以及其他不相关的程序，它就完全看不到了。**



+ **Linux Cgroups**

Namespace 将100 号进程表面上被隔离了起来，但是它所能够使用到的资源（比如 CPU、内存），却是可以随时被宿主机上的其他进程（或者其他容器）占用的。当然，这个 100 号进程自己也可能把所有资源吃光。

Linux Cgroups 就是 Linux 内核中用来为进程设置资源限制的一个重要功能。**Linux Cgroups 的全称是 Linux Control Group。它最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。**

Linux Cgroups 的设计还是比较易用的，简单粗暴地理解呢，它就是一个子系统目录加上一组资源限制文件的组合。而对于 Docker 等 Linux 容器项目来说，它们只需要在每个子系统下面，为每个容器创建一个控制组（即创建一个新目录），然后在启动容器进程之后，把这个进程的 PID 填写到对应控制组的 tasks 文件中就可以了。



### 镜像

Mount Namespace

创建的新进程启用了 Mount Namespace，所以这次重新挂载的操作，只在容器进程的 Mount Namespace 中有效。容器进程启动之前重新挂载它的整个根目录“/”。而由于 Mount Namespace 的存在，这个挂载对宿主机不可见，所以容器进程就可以在里面随便折腾了。

chroot 命令可以帮助你在 shell 中方便地完成这个工作。顾名思义，它的作用就是帮你“change root file system”，即改变进程的根目录到你指定的位置。

为了能够让容器的这个根目录看起来更“真实”，我们一般会在这个容器的根目录下挂载一个完整操作系统的文件系统，比如 Ubuntu16.04 的 ISO。这样，在容器启动之后，我们在容器里通过执行 "ls /" 查看根目录下的内容，就是 Ubuntu 16.04 的所有目录和文件。而这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“**容器镜像**”。它还有一个更为专业的名字，叫作：rootfs（根文件系统）。

需要明确的是，rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像。rootfs 只包括了操作系统的“躯壳”，并没有包括操作系统的“灵魂”。实际上，同一台机器上的所有容器，都共享宿主机操作系统的内核。



Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。联合文件系统（Union File System）的能力。



## 容器编码与Kubernetes作业管理



### Pod

#### 概念

逻辑概念，Kubernetes 真正处理的，还是宿主机操作系统上 Linux 容器的 Namespace 和 Cgroups，而并不存在一个所谓的 Pod 的边界或者隔离环境。



Pod 是 Kubernetes 里的原子调度单位。Kubernetes 项目的调度器，是统一按照 Pod 而非容器的资源需求进行计算的。

容器间紧密协作的超亲密关系：互相之间会发生直接的文件交换、使用 localhost 或者 Socket 文件进行本地通信、会发生非常频繁的远程调用、需要共享某些 Linux Namespace（比如，一个容器要加入另一个容器的 Network Namespace）等等。

Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume。Pod，其实是一组共享了某些资源的容器。

#### Infra容器

镜像：k8s.gcr.io/pause

Pod 的实现需要使用一个中间容器，这个容器叫作 Infra 容器。在这个 Pod 中，Infra 容器永远都是第一个被创建的容器，而其他用户定义的容器，则通过 Join Network Namespace 的方式，与 Infra 容器关联在一起



#### 作用

对于 Pod 里的容器 A 和容器 B 来说：

+ 它们可以直接使用 localhost 进行通信；
+ 它们看到的网络设备跟 Infra 容器看到的完全一样；
+ 一个 Pod 只有一个 IP 地址，也就是这个 Pod 的 Network Namespace 对应的 IP 地址；当然，其他的所有网络资源，都是一个 Pod 一份，并且被该 Pod 中的所有容器共享；
+ Pod 的生命周期只跟 Infra 容器一致，而与容器 A 和 B 无关。



#### Pod字段

凡是调度、网络、存储，以及安全相关的属性，基本上是 Pod 级别的。

这些属性的共同特征是，它们描述的是“机器”这个整体，而不是里面运行的“程序”。比如，配置这个“机器”的网卡（即：Pod 的网络定义），配置这个“机器”的磁盘（即：Pod 的存储定义），配置这个“机器”的防火墙（即：Pod 的安全定义）。更不用说，这台“机器”运行在哪个服务器之上（即：Pod 的调度）。



#### Pod 状态

pod.status.phase，就是 Pod 的当前状态：

+ Pending。这个状态意味着，Pod 的 YAML 文件已经提交给了 Kubernetes，API 对象已经被创建并保存在 Etcd 当中。但是，这个 Pod 里有些容器因为某种原因而不能被顺利创建。比如，调度不成功。

+ Running。这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。（只有一个容器是run）整个pod就是Running 体现. Pod字段 Ready 标识的正常容器的的个数。
+ Succeeded。这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。
+ Failed。这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。
+ Unknown。这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

Pod 对象的 Status 字段，还可以再细分出一组 Conditions：PodScheduled、Ready、Initialized，以及 Unschedulable。它们主要用于描述造成当前 Status 的具体原因

![image2021-12-30_17-39-15](深入剖析 Kubernetes.assets/image2021-12-30_17-39-15.png)



#### 健康检查与恢复机制

##### livenessProbe

Pod 里的容器定义一个健康检查“探针”（Probe）。kubelet 就会根据这个 Probe 的返回值决定这个容器的状态，而不是直接以容器镜像是否运行（来自 Docker 返回的信息）作为依据。这种机制，是生产环境中保证应用健康存活的重要手段。

健康探针可以为 

+ exec 执行命令

  ```
  ...
      livenessProbe:
        exec:
          command:
          - cat
          - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
  ```

+ httpGet http请求

  ```
  ...
  livenessProbe:
       httpGet:
         path: /healthz
         port: 8080
         httpHeaders:
         - name: X-Custom-Header
           value: Awesome
         initialDelaySeconds: 3
         periodSeconds: 3
  ```

  

+ tcpSocket tcp请求

  ```
      ...
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
  ```

  



##### restartPolicy

**当健康检查不通过时，就进行pod重启**， Kubernetes 里的 Pod 恢复机制，也叫 restartPolicy。它是 Pod 的 Spec 部分的一个标准字段（pod.spec.restartPolicy），默认值是 Always，即：任何时候这个容器发生了异常，它一定会被重新创建。

+ Always：在任何情况下，只要容器不在运行状态，就自动重启容器；
+ OnFailure: 只在容器 异常时才自动重启容器；
+ Never: 从来不重启容器。

注意：Pod 的恢复过程，永远都是发生在当前节点上，而不会跑到别的节点上去。事实上，一旦一个 Pod 与一个节点（Node）绑定，除非这个绑定发生了变化（pod.spec.node 字段被修改），否则它永远都不会离开这个节点。这也就意味着，如果这个宿主机宕机了，这个 Pod 也不会主动迁移到其他节点上去。

**若想让pod重新调度到其他node，则需要 工作负载（Deployment）的控制器。**



### 控制器

Pod 对象，其实就是容器的升级版。它对容器进行了组合，添加了更多的属性和字段。控制器直接操作Pod完成kuberneters 的操作。

Kubernetes 架构有一个叫作 kube-controller-manager 的组件。就是一系列控制器的集合。我们可以查看一下 Kubernetes 项目的 pkg/controller 目录：

```
$ cd kubernetes/pkg/controller/
$ ls -d */              
deployment/             job/                    podautoscaler/          
cloud/                  disruption/             namespace/              
replicaset/             serviceaccount/         volume/
cronjob/                garbagecollector/       nodelifecycle/          replication/            statefulset/            daemon/
...
```

**控制器遵循Kubernetes 项目中的一个通用编排模式：控制循环（control loop）**

```

for {
  实际状态 := 获取集群中对象X的实际状态（Actual State）
  期望状态 := 获取集群中对象X的期望状态（Desired State）
  if 实际状态 == 期望状态{
    什么都不做
  } else {
    执行编排动作，将实际状态调整为期望状态
  }
}
```

+ 实际状态：从kuberneters中获取实际状态
+ 期望状态：一般来自于用户提交的Yaml文件

类似 Deployment 这样的一个控制器，实际上都是由**上半部分的控制器定义（包括期望状态），加上下半部分的被控制对象的模板组成的。**



#### Deployment

Pod 的“水平扩展 / 收缩”（horizontal scaling out/in）； 滚动更新

实际上 Deployment 并不实质控制 pod, 而是通过ReplicaSet 控制，实际上是一种“层层控制”的关系。

![711c07208358208e91fa7803ebc73058.webp](深入剖析 Kubernetes.assets/711c07208358208e91fa7803ebc73058.webp.jpg)

##### 水平扩展 / 收缩

Deployment 同样通过“控制器模式”，来修改 ReplicaSet 的控制副本数。**ReplicaSet 负责通过“控制器模式”，保证系统中 Pod 的个数永远等于指定的个数**（比如，3 个）。这也正是 Deployment 只允许容器的 restartPolicy=Always 的主要原因：只有在容器能保证自己始终是 Running 状态的前提下，ReplicaSet 调整 Pod 的个数才有意义。

##### 滚动更新

Deployment 同样通过“控制器模式”，来操作 ReplicaSet 的个数 实现 “滚动更新”。**当pod的模板属性变化时，Deployment 会创建一个新的 ReplicaSet ，逐个替代旧 ReplicaSet 的pod. 将一个集群中正在运行的多个 Pod 版本，交替地逐一升级的过程，就是“滚动更新”。**

**保证服务的连续性**： Deployment Controller 还会确保，在任何时间窗口内，只有指定比例的 Pod 处于离线状态。同时，它也会确保，在任何时间窗口内，只有指定比例的新 Pod 被创建出来。这两个比例的值都是可以配置的，默认都是 DESIRED 值的 25%。

![bbc4560a053dee904e45ad66aac7145d.webp](深入剖析 Kubernetes.assets/bbc4560a053dee904e45ad66aac7145d.webp.jpg)

##### 回滚

kubectl rollout undo 命令，就能把整个 Deployment 回滚到上一个版本：

```
$ kubectl rollout undo deployment/nginx-deployment
deployment.extensions/nginx-deployment
```

若要回退更前的版本，我需要使用 kubectl rollout history 命令，查看每次 Deployment 变更对应的版本

```

$ kubectl rollout history deployment/nginx-deployment
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl create -f nginx-deployment.yaml --record
2           kubectl edit deployment/nginx-deployment
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.91

# 指定版本回退

$ kubectl rollout undo deployment/nginx-deployment --to-revision=2
deployment.extensions/nginx-deployment
```



#### StatefulSet

StatefulSet: **有状态应用**

在分布式场景中，多个实例之间，往往有依赖关系，比如：主从关系、主备关系。或者数据存储类应用，它的多个实例，往往都会在本地磁盘上保存一份数据。而这些实例一旦被杀掉，即便重建出来，实例与数据之间的对应关系也已经丢失，从而导致应用失败。

这种实例之间有不对等关系，以及实例对外部数据有依赖关系的应用，就被称为“有状态应用”（Stateful Application）。	

##### 状态

+ 拓扑状态

  应用的多个实例之间不是完全对等的关系。这些应用实例，**必须按照某些顺序启动**，比如应用的主节点 A 要先于从节点 B 启动。而如果你把 A 和 B 两个 Pod 删除掉，它们再次被创建出来时也必须严格按照这个顺序才行。并且，新创建出来的 Pod，**必须和原来 Pod 的网络标识一样**，这样原先的访问者才能使用同样的方法，访问到这个新 Pod。

+ 存储状态

  **应用的多个实例分别绑定了不同的存储数据。**对于这些应用实例来说，**Pod A 第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间 Pod A 被重新创建过。**这种情况最典型的例子，就是一个数据库应用的多个存储实例。

+ 1

**StatefulSet 的核心功能，就是通过某种方式记录这些状态，然后在 Pod 被重新创建时，能够为新 Pod 恢复这些状态。**



##### Headless Service

Service 是 Kubernetes 项目中用来将一组 Pod 暴露给外界访问的一种机制。用户访问Service,就可以访问到某个具体的 Pod。

+ Service VIP

  Virtual IP，即：虚拟 IP；当我访问 10.0.23.1 这个 Service 的 IP 地址时，10.0.23.1 其实就是一个 VIP，它会把请求转发到该 Service 所代理的某一个 Pod 上。

+ Service DNS

  只要我访问“my-svc.my-namespace.svc.cluster.local”这条 DNS 记录，就可以访问到名叫 my-svc 的 Service 所代理的某一个 Pod。

  + Normal Service

    Normal Service，访问“my-svc.my-namespace.svc.cluster.local”解析到的，正是 my-svc 这个 Service 的 VIP，后面的流程就跟 VIP 方式一致了。

  + Headless Service

    访问“my-svc.my-namespace.svc.cluster.local”解析到的，直接就是 my-svc 代理的某一个 Pod 的 IP 地址。可以看到，这里的区别在于，**Headless Service 不需要分配一个 VIP，而是可以直接以 DNS 记录的方式解析出被代理 Pod 的 IP 地址。**

    ```
    
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx
    ```

    **标准Service模板Yaml. 但 clusterIP 字段的值是：None**

    Headless 所代理的所有 Pod 的 IP 地址，都会被绑定一个这样格式的 DNS 记录。 只要你知道了一个 Pod 的名字，以及它对应的 Service 的名字，你就可以非常确定地通过这条 DNS 记录访问到 Pod 的 IP 地址。

    ```
    <pod-name>.<svc-name>.<namespace>.svc.cluster.local
    ```



##### 创建pod

StatefulSet 给它所管理的所有 Pod 的名字，进行了编号，编号规则是：

```
<statefulset name>-<ordinal index>
```

这些编号都是从 0 开始累加，与 StatefulSet 的每个 Pod 实例一一对应，绝不重复。

更重要的是，这些 Pod 的创建，也是**严格按照编号顺序进行的。比如，在 web-0 进入到 Running 状态、并且细分状态（Conditions）成为 Ready 之前，web-1 会一直处于 Pending 状态。**

Kubernetes 中的StatefulSet 就成功地将 **Pod 的拓扑状态（比如：哪个节点先启动，哪个节点后启动），按照 Pod 的“名字 + 编号”的方式固定了下来。**此外，Kubernetes 还为每一个 Pod 提供了一个固定并且唯一的访问入口，即：这个 Pod 对应的 DNS 记录。

这些状态，**在 StatefulSet 的整个生命周期里都会保持不变，绝不会因为对应 Pod 的删除或者重新创建而失效。需要新建或者删除 Pod 进行“调谐”的时候，它会严格按照这些 Pod 编号的顺序，逐一完成这些操作。**



##### 存储状态

```

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.1
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```

StatefulSet 额外添加了一个 volumeClaimTemplates 字段。被这个 StatefulSet 管理的 Pod，都会声明一个对应的 PVC；而这个 PVC 的定义，就来自于 volumeClaimTemplates 这个模板字段。更重要的是，这个 PVC 的名字，会被分配一个与这个 Pod 完全一致的编号。`<PVC 名字 >-<StatefulSet 名字 >-< 编号 >`

当StatefulSet 控制的pod 删除一个（web-0）后，新的 web-0 Pod 被创建出来之后，Kubernetes 为它查找名叫 www-web-0 的 PVC 时，就会直接找到旧 Pod 遗留下来的同名的 PVC，进而找到跟这个 PVC 绑定在一起的 PV。

**通过这种方式，Kubernetes 的 StatefulSet 就实现了对应用存储状态的管理。**

##### 总结

StatefulSet 的控制器直接管理的是 Pod。StatefulSet 里的不同 Pod 实例，不再像 ReplicaSet 中那样都是完全一样的，而是有了细微区别的。而 StatefulSet 区分这些实例的方式，就是通过在 Pod 的名字里加上事先约定好的编号。

**有了这个编号后，StatefulSet 就使用 Kubernetes 里的两个标准功能：Headless Service 和 PV/PVC，实现了对 Pod 的拓扑状态和存储状态的维护。**



#### DaemonSet

##### 特征

+ 这个 Pod 运行在 Kubernetes 集群里的每一个节点（Node）上；

+ 每个节点上只有一个这样的 Pod 实例；

+ 当有新的节点加入 Kubernetes 集群后，该 Pod 会自动地在新节点上被创建出来；而当旧节点被删除后，它上面的 Pod 也相应地会被回收掉。

适用一些插件：

+ 各种网络插件的 Agent 组件，都必须运行在每一个节点上，用来处理这个节点上的容器网络；

+ 各种存储插件的 Agent 组件，也必须运行在每一个节点上，用来在这个节点上挂载远程存储目录，操作容器的 Volume 目录；

+ 各种监控组件和日志组件，也必须运行在每一个节点上，负责这个节点上的监控信息和日志搜集。



##### DaemonSet Controller

从 Etcd 里获取所有的 Node 列表，然后遍历所有的 Node。这时，它就可以很容易地去检查，当前这个 Node 上是不是有一个携带了key=value 标签的 Pod 在运行。

+ 没有这种 Pod，那么就意味着要在这个 Node 上创建这样一个 Pod；
+ 有这种 Pod，但是数量大于 1，那就说明要把多余的 Pod 从这个 Node 上删除掉；
+ 正好只有一个这种 Pod，那说明这个节点是正常的。



如何在指定的node 上运行一个pod

+ nodeSelector 与 nodeName

```
nodeSelector:
    name: <Node名字>
```

+ nodeAffinity  与 Toleration

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
            - key: metadata.name
              operator: In
              values:
              - node-geektime
  ```

  ```
  
  apiVersion: v1
  kind: Pod
  metadata:
    name: with-toleration
  spec:
    tolerations:
    - key: node.kubernetes.io/unschedulable
      operator: Exists
      effect: NoSchedule
  ```



**节点亲和性**

DaemonSet Controller 会在创建 Pod 的时候，自动在这个 Pod 的 API 对象里，加上这样一个 nodeAffinity 定义。DaemonSet 并不需要修改用户提交的 YAML 文件里的 Pod 模板，而是在向 Kubernetes 发起请求之前，直接修改根据模板生成的 Pod 对象。

- requiredDuringSchedulingIgnoredDuringExecution

  硬需求，*必须*满足的规则与nodeSelector一致

  如果你指定了多个与 `nodeAffinity` 类型关联的 `nodeSelectorTerms`，则 **如果其中一个** `nodeSelectorTerms` 满足的话，pod将可以调度到节点上。

  如果你指定了多个与 `nodeSelectorTerms` 关联的 `matchExpressions`，则 **只有当所有** `matchExpressions` 满足的话，Pod 才会可以调度到节点上。

- preferredDuringSchedulingIgnoredDuringExecution

  调度器将尝试执行但不能保证的*偏好*，如果无法满足，可将pod调度到其他项目中去

  weight 范围是 1-100。 对于每个符合所有调度要求（资源请求、RequiredDuringScheduling 亲和性表达式等） 的节点，调度器将遍历该字段的元素来计算总和，并且如果节点匹配对应的 MatchExpressions，则添加“权重”到总和。 然后将这个评分与该节点的其他优先级函数的评分进行组合。 总分最高的节点是最优选的。

注： IgnoredDuringExecution 意味着已调度到node上后，node运行中发生改变，不再满足于pod的亲和性规则，pod可在该node上继续运行。



**污点与容忍度**

在 Kubernetes 项目中，当一个节点的网络插件尚未安装时，这个节点就会被自动加上名为node.kubernetes.io/network-unavailable的“污点”。而通过这样一个 Toleration，调度器在调度这个 Pod 的时候，就会忽略当前节点上的“污点”，从而成功地将网络插件的 Agent 组件调度到这台机器上启动起来。

[*节点亲和性*](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) 是 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 的一种属性，它使 Pod 被吸引到一类特定的[节点](https://kubernetes.io/zh/docs/concepts/architecture/nodes/) （这可能出于一种偏好，也可能是硬性要求）。 *污点*（Taint）则相反——它使节点能够排斥一类特定的 Pod。

容忍度（Toleration）是应用于 Pod 上的，允许（但并不要求）Pod 调度到带有与之匹配的污点的节点上。

污点和容忍度（Toleration）相互配合，可以用来避免 Pod 被分配到不合适的节点上。 每个节点上都可以应用一个或多个污点，这表示对于那些不能容忍这些污点的 Pod，是不会被该节点接受的。

