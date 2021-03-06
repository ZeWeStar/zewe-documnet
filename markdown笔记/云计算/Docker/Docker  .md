## Docker  

#### Docker 架构

通过docker client 连接 dockerd ,使用restful接口 ，（client 与 dockerd 可不在同一台机器上）

![1576412720164](Docker  .assets\1576412720164.png)



#### Docker Engine - CS结构

![1576412471222](Docker  .assets\1576412471222.png)



#### Docker image



![1576415268592](Docker  .assets\1576415268592.png)

#### Docker container



![1576417202174](Docker  .assets\1576417202174.png)



#### Dockerfile

**1、 FROM**  

base image, 使用官方image做base image, [也**可引用 本地已有的 images**]

![1576503149349](Docker  .assets\1576503149349.png)

**2、LABEL** 

注释  descs

![1576503427380](Docker  .assets\1576503427380.png)

**3、RUN**

命令行执行命令。避免无用分层，建议合并多条命令为一行。且使用反斜线为换行符，提高可读性

![1576503689672](Docker  .assets\1576503689672.png)

**4、WORKDIR**

设定当前工作目录。替换 RUN cd ，尽量使用绝对目录

![1576503823292](Docker  .assets\1576503823292.png)

**5、ADD and COPY**

添加当前系统文件/目录到 image 指定目录 ，ADD 可自动解压缩。添加远程文件/目录 使用 RUN curl 或 wget

![1576504159420](Docker  .assets\1576504159420.png)

**6、ENV**

设置常量

![1576504823751](Docker  .assets\1576504823751.png)

**7、RUN**

执行命令，并建立新的 Image Layer



**8、CMD**

设置容器启动后默认执行的命令和参数，多条时最后一条会执行。



**9、ENTRYPOINT** 

设置容器启动时的运行的命令，让容器以应用程序或服务的形式运行，一般设置启动线程，一定会执行。

接收外部参数的形式 例如 Dockerfile ,可以在 run container 时后接参数

```
ENTRYPOINT ["/usr/bin/xxx"]
CMD []
```



#### network



![1577626027211](Docker  .assets\1577626027211.png)



![1577716700305](Docker  .assets\1577716700305.png)





#### **network namespace**



+ 添加 linux namespace  

  sudo ip netns add xxx   # sudo ip netns add test1

+ 查看 linus namespace

  sudo ip netns list

+ 新建一对 Veth pair 接口

  sudo ip link add veth-test1 type veth peer name veth-test2

+ 将 link 口添加到 namespace中

  sudo ip link set veth-test1 netns test1

+ 给namespace 中 link 定义 ip地址

  sudo ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1

+ 启动namespace 中 link

  sudo ip netns exec test1 ip link set dev veth-test1 up

+ 



#### 镜像命令

+ build

```
docker build -t 172.16.5.172/bobft/bobft-cce-web-server:V2.0-R0903-202109171653 .
```

+ 登录

```
docker login 172.16.5.172
```

+ push

```
docker push 172.16.5.172/bobft/bobft-cce-api-server:V2.0-R0903-202109171653
```

+ 压缩

```
docker save 172.16.5.172/bobft/bobft-cce-openapi-server:V1.0.0-202109231212 | gzip > bobft-cce-openapi-server_V1.0.0-202109231212.tar.gz
```



maven 路由

```
解决方案
一、在虚机上添加到172.17.3.0/24段的路由
route add -net 172.17.3.0 netmask 255.255.255.0 gw 172.16.5.254 dev ens192
 
设置开机自动添加路由
编辑/etc/rc.local，
vim /etc/rc.local
添加
route add -net 172.17.3.0 netmask 255.255.255.0 gw 172.16.5.254 dev ens192
 
保存退出:wq
chmod +x /etc/rc.local
```



#### docker 基础命令

+ service docker start # 启动

+ service docker restart # 重启

+ service docker stop # stop

+ docker pull ubuntu:14.04    # 从 registry 获取 image

+ docker images || docker image ls  # 查看image 列表

+ docker history <imageid> # 查看image 层数

+ docker run zewe/hello-world # 启动image

+ docker container ls [-a] || docker ps # 当前运行的容器

+ docker run -it ubuntu:14.04 # 交互式

+ docker run -d xxx  #后台执行

+ docker image rm <id> || docker rmi <id> # remove image

+ docker container rm <id> # remove

+ docker rm $(docker container ls -aq) # 删除所有container

+ docker rm $(docker container ls -f  "status=exited"  -q)  # 删除所有exited 状态的容器 ，f:filter

+ docker container  commit || docker commit <names> [repository:tag]

  #container to image 例： docker commit hungry_darwin zewe/centos：latest

+ docker build  # Dockerfile 例 docker build -t zewe/centos-vim:latest . 

+ docker commit  <container id> image-name:latest  #cretae image

+ docker login #登录

+ docker push zewe/hello-world:latest # 推送镜像到 docker hub

+ docker stop xxx # 停止容器

+ docker start xxx #启动容器

+ docker inspect  <containerid> # 返回容器信息

+ docker logs <xxxid> #执行命令信息 

+ docker network xxx # docker 网络 -help 

+ docker exec  -it <container> [commend] # 容器执行命令 # 例 docker exec -it xxxx ip a 

+  docker inspect <name:latest>  #详细内容

 