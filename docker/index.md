# Docker常用命令

<!--more-->
## 服务相关
### 启动 docker 服务
```shell
systemctl start docker
```
### 停止 docker 服务
```shell
systemctl stop docker
```
### 重启 docker 服务
```shell
systemctl restart docker
```
### 查看 docker 服务状态
```shell
systemctl status docker
```
### 设置开机启动 docker 服务
```shell
systemctl enable docker
```

## 镜像相关
### 搜索镜像
```shell
 docker search nginx
```
### 拉取镜像
```shell
 docker  pull nginx
```
### 构建镜像
```shell
 docker build -t my_image:1.0 .
```
### 查看本地镜像
```shell
 docker images
 docker images -q//查看本地镜像 id
```
### 删除本地镜像
```shell
 docker rmi mysql:5.7
 docker rmi `docker images -q` //全部删除本地镜像
```
### 导出镜像
```shell
 docker save -o image.tar target_image:tag
```
### 导入镜像
```shell
 docker load -i image.tar
```
### 给镜像打标签
```shell
 docker tag image_id new_image_name:tag
```
## 容器相关
### 创建容器
```shell
 docker run -d --name=my_container -p 8080:8080 tomcat:latest
```
### 查看容器列表
```shell
 docker ps
 docker ps -q //查看正在运行的容器ID列表
```
### 停止容器
```shell
 docker stop container
```
### 启动已停止的容器
```shell
 docker start my_container
```
### 删除容器
```shell
 docker rm my_container
```
### 进入容器
```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```
其中，OPTIONS 可以指定一些参数，CONTAINER 是容器的名称或 ID，COMMAND 是要执行的命令，ARG 是命令的参数。

举例：
```shell
docker exec -it my_container sh -c "echo Hello World && ls -l"
```
### 查看容器信息
```shell
 docker inspect my_container
```
### 数据拷贝
```shell
 docker cp container_id:/app/html /mnt/ //将容器中 /app/html 目录拷贝到宿主机 /mnt/ 目录中
 docker cp /mnt/dist container_id:/app/ //将宿主机 /mnt/dist 目录拷贝到容器的 /app 目录中
```
