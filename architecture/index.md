# Docker


## Docker架构及底层技术
### Docker Engine
后台进程（dockerd）
Rest Api Server
CLI接口

### 架构
![docker_arch.png](/images/docker_arch.png)

### 底层技术支持
Namespaces：做隔离pid,net,ipc,mnt,uts
Control groups: 做资源限制
Union file systems：Container和 image的分层

## Docker Image
### 定义
文件和meta data的集合
分层的，并且每一层都可以添加改变删除文件，成为一个新的 image
不同的 image可以共享相同的 layer
本身是 read-only的
### 获取
Build from Dockerfile
Pull from Registry
## Docker Container
### 定义
通过 Image创建
在 Image layer之上建立一个 container layer
类比面向对象：类和实例
Image负责 app的存储和分发，Container负责运行 app
### 构建自己的镜像
```
docker commit container_name repositry_name/tag_name 
```

## Dockerfile语法梳理及最佳实践
### From
尽量使用官方的 image
```
FROM scratch # 制作 base image
FROM centos # 使用 base image
FROM ubuntu:14.04
```
### Label
```
LABEL maintainer="email.com"
LABEL version="1.0"
LABEL description="this is description"
```
### RUN
复杂的 RUN请用反斜线换行
避免无用分层，合并多条命令成一行
```
RUN yum update && yum install -y vim \
    python-dev
```
### WORKDIR
不要使用 RUN cd
```
WORKDIR /root
WORKDIR /test #如果没有会自动创建 test目录
```
### ADD and COPY
COPY优于 ADD
ADD 除了 COPY还有额外的功能（解压）
添加远程文件/目录请使用 curl或者 wget
```
ADD hello /
ADD test.tar.gz / # 添加到根目录并解压
WORDIR /root
COPY hello test/
```
### ENV
尽量使用 ENV
```
ENV MYSQL_VERSION 5.6
RUN apt-get install -y mysql-server= "${MYSQL_VERSION}"
```
### VOLUME and EXPOSE

### CMD and ENTRYPOINT
* CMD 

设置容器启动时默认执行的命令和参数

如果 doker run制定了其他命令，CMD命令将被忽略

如果定义了多个 CMD,只有最后一个会执行
* ENTRYPOINT 

设置容器启动时运行的命令

让容器以应用程序或者服务的形式运行

不会被忽略，一定会执行

最佳实践：写一个 shell脚本作为 entrypoint
#### Shell 格式
```
RUN apt-get install -y vim 
CMD echo "hello docker"
ENTRYPOINT echo "hello docker"
```
#### Exec 格式
```
RUN ["apt-get", "install", "-y", "vim"]
CMD ["/bin/echo", "hello docker"]
ENTRYPOINT ["/bin/echo", "hello docker"]
```
### 最佳实践
```
FROM python:2.7
LABEL "maintaioner=mail.com"
RUN pip install flask
COPY app.py /app/
WORKDIR /app
EXPOSE 5000
CMD ["python", "app.py"]
```
## 镜像的发布
```
docker login
docker push dockerhub_id/name
```
推荐的做法是跟 github做关联
## Docker Network
### 网络的分层
ISO/OSI 7层
TCP/IP 5 层
#### 公有 ip和私有 ip
Public IP：可以访问 Internet
Private IP:不可访问 internet
* A类 10.0.0.0--10.255.255.255
* B类 172.16.0.0--172.31.255.255
* C类 192.168.0.0--192.168.255.255
#### 网络地址转换 NAT
#### ping和 telnet
* ping（ICMP）:验证 ip的可达性
* telnet:验证服务的可用性
#### wireshark

### Linux网络命名空间
主机和容器网络隔离，容器具有自己独立的网络命名空间，且他们也可以互通
```
ip netns add test1
ip netns add test2
ip netns list
ip netns exec test1 ip a
ip netns exec test1 ip link set dev lo up
ip link add veth-test1 type veth peer name veth-test2
ip link set veth-test1 netns test1
ip link set veth-test2 netns test2
ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1
ip netns exec test2 ip addr add 192.168.1.2/24 dev veth-test2
ip netns exec test1 ip link set dev veth-test1 up
ip netns exec test2 ip link set dev veth-test2 up
ip netns exec test1 ping 192.168.1.2
ip netns exec test2 ping 192.168.1.1
```
![network_namespace.png](/images/network_namespace.png)
### Bridge0详解
![bridge_network.png](bridge_network.png)
### 容器之间的 link
一个容器如果想访问到另外一个容器，可以通过link的方式，知道对方的network name即可访问
link是有方向限制的
如果是连接到用户自定义的 bridge network，默认两者网络都已关联，没有方向限制
### 单机
#### Bridge Network
```
docker network create -d bridge my-bridge
brctl show #可以看到创建好的 bridge
docker run -d --name test --network my-bridge busybox
docker network connect my-bridge test
```
#### Host Network
host模式类似于Vmware的桥接模式，与宿主机在同一个网络中，但没有独立IP地址。
```
docker run --name=nginx --net=host -p 80:80 -d nginx
```
#### None Network
容器有自己的 network namespace，但是没有做任何的网络配置。
### 多机
#### Overlay Network
```
docker network create -d overlay demo
```

## Docker的持久化存储和数据共享
### Data Volume
![volume_arch.png](/images/volume_arch.png)
```
docker run -v mysql:/var/lib/mysql
```
### Bind Mouting
```
docker run -v /home/aaa:/root/aaa
```
### 持久化方案
* 基于本地文件系统的 Volume
* 基于 plugin的 Volume,例如 NAS,aws
### Volume 类型
* 受管理的 data Volume，由 docker后台自动创建
* 绑定挂载的 Volume,具体挂载位置可以由用户指定
## Docker Compose
### 定义
Docker Compose是一个工具，可以通过一个 yml文件定义多容器的 docker应用
### 三大概念
#### Services
一个 service代表一个 container
Service的启动类似 docker run,可以给其指定 network和 volume

#### Networks
#### Volumes
### 水平扩容和负载均衡
```
docker-compose up --scale web=2 -d
```
负载均衡

```yaml
lb:
  image: dockercloud/haproxy
  links:
    - web
  ports:
    8080:80
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
```
## 容器编排kubenetes
### ReplicaSet和 Replication Controller
### Deployment
```
kubectl create -f deployment.yml
kubectl get deployment -o wide
##升级
kubectl set image deployment nginx-deployment nginx=nginx:1.13
kubectl rollout history deployment nginx-deployment
##回滚
kubectl rollout undo deployment nginx-deployment
```
{{<admonition tip>}}
自动补全：
```
kubectl completion zsh
source <(kubectl completion zsh)
```
{{</admonition>}}

### Services
### ClusterIP
pod改变，service的 clusterIP不会改变

内部使用
### NodePort
```
kubectl expose pods nginx-pod --type=NodePort
```
有端口范围限制

### LoadBalancer
```
kubectl expose pods nginx-pod --type=LoadBalancer
```
### External name
