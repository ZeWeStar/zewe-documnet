# 深入剖析 Kubernetes

## 基础概念

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

+ Running。这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。
+ Succeeded。这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。
+ Failed。这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。
+ Unknown。这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

Pod 对象的 Status 字段，还可以再细分出一组 Conditions：PodScheduled、Ready、Initialized，以及 Unschedulable。它们主要用于描述造成当前 Status 的具体原因

