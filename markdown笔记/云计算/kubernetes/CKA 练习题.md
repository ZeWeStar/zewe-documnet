# CKA 练习题



## RBAC

1、创建一个新的 ClusterRole 并将其绑定到范围为特定 namespace 的特定  ServiceAccount 

创建一个名字为 **deployment-clusterrole** 且仅允许**创建**以下资源类型的新 **ClusterRole**： 

+ Deployment 

+ StatefulSet 

+ DaemonSet 

在现有的 namespace **app-team1** 中创建有个名为 **cicd-token** 的新 **ServiceAccount**。 

限 于 namespace app-team1 ， 将 新 的 ClusterRole deployment-clusterrole 绑 定 到 新 的 

ServiceAccount cicd-token。



```
# 创建namespace
kubectl create ns app-team1

# 创建 ClusterRole
kubectl create clusterrole deployment-role --verb=create --resource=Deployment,StatefulSet,DaemonSet

# 创建 ServiceAccount
kubectl create serviceaccount cicd-token -n app-team1

# 创建 rolebinding
# 注意 serviceaccount must be <namespace>:<name>
kubectl create rolebinding deployment-role-cicd-token-bind --clusterrole=deployment-role --serviceaccount=cicd-token -n app-team1

# 查看 binding
kubectl describe rolebinding deployment-role-cicd-token-bind -n app-team1
```



## 调度

1、设置 node 不可用

+ 设置某节点不可调度

  影响最小，只会将node调为SchedulingDisabled之后再发创建pod，不会被调度到该节点旧有的pod不会受到影响，仍正常对外提供服务

  ```
  kubectl cordon ek8s-node-1
  ```

+ 命令node节点开始释放所有pod，并且不接收新的pod进程

  ```
   kubectl drain ek8s-node-1 --delete-local-data --ignore-daemonsets --force
  ```

  + delete-local-data[=false]: 删除pod中的 empty dir 存在的本地数据
  + ignore-daemonsets[=false]：忽略daemonset的 pod
  + force[=false]: 强制





## 网络



