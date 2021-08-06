# Istio

## Kubernetes相关

### 容器

#### 容器运行时

+ Docker

+ Podman ：完全兼容docker，无daemon 、集成入OS中，Redhat推出
+ RKT
+ containerd (k8s推出)



### 工作负载

#### Deployment 演进

+ replicationcontroller  - 不支持set标签选择，只支持等于
+ replicaset - 支持set标签
+ Deployment -  replicaset的管理



#### 滚动升级

滚动升级：工作负载层面的升级，通过创建新的工作负载代替旧的工作负载



### 服务发现

集群内部/集群外部

容器内 http://servicename:serviceport -> pod:targetPort

+ headless 类型,创建关联Pod的一条DNS记录

![image-20210618135533879](Istio.assets/image-20210618135533879.png)

![image-20210618135658164](Istio.assets/image-20210618135658164.png)



## Spring Boot

Spring Actuato 可监控信息项 - 配置信息，beans(依赖注入), conditions(阅读源码-SpringBoot 整合bean)

![image-20210618162014630](Istio.assets/image-20210618162014630.png)



镜像加速器

![image-20210618170038968](Istio.assets/image-20210618170038968.png)

使用私服镜像进行编排

![image-20210618192732142](Istio.assets/image-20210618192732142.png)

## Istio

### 版本

![image-20210618212250214](Istio.assets/image-20210618212250214.png)

### 场景

#### 超时、限流、断路器

![image-20210618203535767](Istio.assets/image-20210618203535767.png)

如何不修改代码，实现上述场景？ - Istio sidecar

![image-20210618204509261](Istio.assets/image-20210618204509261.png)

![image-20210618204607470](Istio.assets/image-20210618204607470.png)

### pilot 

istio的注册发现中心

![image-20210619091102934](Istio.assets/image-20210619091102934.png)

### 安装

#### pod注入Envoy

+ 单个手动注入

![image-20210619091938287](Istio.assets/image-20210619091938287.png)

istioctl kube-inject -f  xxx.yaml

istioctl 给指定yaml添加sidecar  (Envoy), 修改自写的yaml文件



kubectl apply -f <(istioctl kube-inject -f user.yaml) -n ns-czw



+ 指定namespace 批量操作添加sidecar

  ![image-20210619093230547](Istio.assets/image-20210619093230547.png)

  + 1 



启动时

init 启动时修改iptables 劫持入口与出口流量

![image-20210619092213460](Istio.assets/image-20210619092213460.png)



### 案例

VirtualService 配置规则控制流量

![image-20210619101146567](Istio.assets/image-20210619101146567.png)





![image-20210619102421949](Istio.assets/image-20210619102421949.png)



![image-20210619102732883](Istio.assets/image-20210619102732883.png)

![image-20210619102827737](Istio.assets/image-20210619102827737.png)







![image-20210619150203595](Istio.assets/image-20210619150203595.png)