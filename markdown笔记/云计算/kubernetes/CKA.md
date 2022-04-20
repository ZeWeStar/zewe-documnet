# CKA



## 01-RBAC

创建一个新的 ClusterRole 并将其绑定到范围为特定 namespace 的特定  ServiceAccount 

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



## 02-设置节点不可调度

将ek8s-node-1节点设置为不可用，然后重新调度该节点上的所有Pod

```
# 标记节点 ek8s-node-1 不可调用
kubectl cordon ek8s-node-1

# 驱逐节点上的pod（包含 empty-dir 的pod,但忽略 daemonset pod）; 
kubectl drain ek8s-node-1 --delete-emptydir-data --ignore-daemonsets

# 查看节点
kubectl get node
```



## 03-升级 Kuberneters 节点

建 admin 节点 1.22.x升级到1.23.x，升级包括 kubectl、kubelet;  不包括 etcd 、work node、CNI、DNS 

```
# 腾空节点：标记节点不可用，驱逐节点上pod
kubectl cordon ek8s-node-1
kubectl drain ek8s-node-1 --delete-emptydir-data --ignore-daemonsets

# 登录 ek8s-node-1 节点 
ssh ek8s-node-1

# 查找在列表中查找最新的 1.23.1 版本 
apt update
apt-cache madison kubeadm

# 升级 kubeadm, 如果查到 1.23.1
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.23.1-00 && \
apt-mark hold kubeadm

# 验证 kubeadm 版本，查看kubeadm 升级计划
kubeadm version
kubeadm upgrade plan

# 升级master节点，注意：忽略 etcd 
sudo kubeadm upgrade apply v1.23.1 --etcd-upgrade=false

#升级kubectl 和 kubelete
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.23.1-00 kubectl=1.23.1-00 && \
apt-mark hold kubelet kubectl

# 重启 kubelet 
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# 解除节点不可调度
kubectl uncordon ek8s-node-1
```



## 04-Etcd数据备份恢复

针对etcd实例https://127.0.0.1:2379创建一个快照，保存到/srv/data/etcd-snapshot.db。在创建快照的过程中，如果卡住了，就键入ctrl+c终止，然后重试。
然后恢复一个已经存在的快照： /var/lib/backup/etcd-snapshot-previous.db
执行etcdctl命令的证书存放在：
ca证书：/opt/KUIN00601/ca.crt
客户端证书：/opt/KUIN00601/etcd-client.crt
客户端密钥：/opt/KUIN00601/etcd-client.key

```
# 创建卷快照
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/opt/KUIN00601/ca.crt --cert=/opt/KUIN00601/etcd-client.crt --key=/opt/KUIN00601/etcd-client.key \
  snapshot save /srv/data/etcd-snapshot.db
  
# 指定快照恢复etcd
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/opt/KUIN00601/ca.crt --cert=/opt/KUIN00601/etcd-client.crt --key=/opt/KUIN00601/etcd-client.key \
  snapshot restore /var/lib/backup/etcd-snapshot-previous.db
```



## 05-Networkpolicy

创建一个名字为all-port-from-namespace的NetworkPolicy，这个NetworkPolicy允许my-app命名空间下的Pod访问fubar 命名空间下的开放的 **80 端口**。且 **fubar  只允许 my-app 命名空间下的pod访问**

```
# 创建 ns my-app、fubar
kubectl create ns my-app
kubectl create ns fubar

# 查看 ns 与其lables
kubectl get ns --show-labels

# 创建 networkpolicy , vim all-port-from-namespace.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: all-port-from-namespace
  namespace: fubar
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: my-app
    ports:
    - protocol: TCP
      port: 80
  
  
# 执行yaml
kubectl apply -f all-port-from-namespace.yaml -n fubar
  
# 查看
kubectl describe networkpolicy all-port-from-namespace -n fubar
      
```



## 06-创建SVC

重新配置一个已经存在的deployment front-end，在名字为nginx的容器里面添加一个端口配置，名字为http，暴露端口号为80，然后创建一个service，名字为front-end-svc，暴露该deployment的http端口，并且service的类型为NodePort。

```
# deploy container 暴露端点
kubectl edit deploy front-end -n default

#添加ports字段
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
          
# 创建svc
kubectl expose deployment front-end --port=80 --target-port=80 --type=NodePort --name=front-end-svc
```



## 07-创建Ingress

在ing-internal命名空间下创建一个ingress，名字为ping，代理的service hello，端口为5678，配置路径/hello。
验证：访问curl -kL <INTERNAL_IP>/hi会返回hi

```
# yaml
vim 07-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ping
  namespace: ing-internal
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /hello
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              number: 5678

# 执行
kubectl apply -f 07-ingress.yaml
```



## 08-扩展 deployment副本数

扩容名字为guestbook的deployment的副本数为6

```
# 扩展
kubectl scale deployment guestbook --replicas=6

# 查看
kubectl get deploy guestbook -n default
```



## 09-将 pod 部署到指定 node 节点上

创建一个Pod，名字为nginx-kusc00401，镜像地址是nginx，调度到具有disk=spinning标签的节点上

```
# 节点设置标签（已有标签不用设置）
kubectl label nodes ek8s-node-1 disk=spinning

# 查看标签
kubectl get node --show-labels

# 创建 09-pod.yaml
vim 09-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00401
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disk: spinning
    

# 执行
kubectl apply -f 09-pod.yaml

```



## 10-检查有多少 node 节点是健康状态

检查集群中有多少节点为Ready状态，并且去除包含NoSchedule污点的节点。之后将数字写到/opt/KUSC00402/kusc00402.txt

```
#查看node状态，计算ready的node数
kubectl get nodes | grep ready

#查看有污点的node数
kubectl  describe node | grep Taint | grep NoSchedule

#两数相减记入指定文件
echo n >> /opt/KUSC00402/kusc00402.txt
```



## 11-创建包含多个 container 的 Pod

创建一个Pod，名字为kucc1，这个Pod可能包含1-4容器，该题为四个：nginx+redis+memcached+consul

```
# vim 10-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kucc1
  namespace: default
spec:
  containers:
  - name: nginx
    image: nignx
  - name: redis
    image: redis
  - name: memcached
    image: memcached
  - name: consul
    image: consul
  
# 执行
kubectl apply -f 10-pod.yaml

#检查pod状态
kubectl get pod kucc1 -o wide
```



## 12-创建PV

创建一个pv，名字为app-config，大小为2Gi，访问权限为ReadWriteMany。Volume的类型为hostPath，路径为/srv/app-config

```
# vim 12-pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-config
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/srv/app-config"
    
# 执行
kubectl apply -f 12-pv.yaml

# 检查
kubectl describe pv app-config 
```



## 13-创建 PVC并挂载

创建一个名字为pv-volume的pvc，指定storageClass为csi-hostpath-sc，大小为10Mi
然后创建一个Pod，名字为web-server，镜像为nginx，并且挂载该PVC至/usr/share/nginx/html，挂载的权限为ReadWriteOnce。之后通过kubectl edit或者kubectl path将pvc改成70Mi，并且记录修改记录。

```
# vim 13-pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  storageClassName: csi-hostpath-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi

# 执行
kubectl apply -f 13-pvc.yaml


# vim 13-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  volumes:
    - name: pv-volume
      persistentVolumeClaim:
        claimName: pv-volume
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-volume

# 执行 
kubectl apply -f 13-pod.yaml

# 扩容 注意记录
kubectl edit pvc pv-volume --record ##此处一定注意别丢了记录


```



## 14-监控 pod 的日志筛选指定错误到文件

查看日志并筛选到指定文件

```
 kubectl logs foobar |grep unable-to-access-website > /opt/KUTR00101/foobar
```

