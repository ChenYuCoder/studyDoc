# Docker学习文档
---
* 搜索镜像：https://hub.docker.com
* 下载镜像：docker pull centos:centos7
* 本地已存在镜像：docker images
* HelloWorld：docker run centos:centos7 /bin/echo "Hello World!"
---
* 输出全部命令：docker
* 输出具体命令帮助：docker command --help
* 查看当前docker容器：docker ps
  * -a :展示全部
>  * CONTAINER ID: 容器 ID。
>  * IMAGE: 使用的镜像。
>  * COMMAND: 启动容器时运行的命令。
>  * CREATED: 容器的创建时间。
>  * STATUS: 容器状态。
>  * 状态有7种：
>    * created（已创建）
>    * restarting（重启中）
>    * running（运行中）
>    * removing（迁移中）
>    * paused（暂停）
>    * exited（停止）
>    * dead（死亡）
>  * PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
>  * NAMES: 自动分配的容器名称。

* 显示容器输出：docker logs containerId
* 终止容器输出：docker stop containerId
* 启动已停止容器：docker start containerId

* 运行容器：docker run image command
  * -i: 允许你对容器内的标准输入 (STDIN) 进行交互
  * -t: 在新容器内指定一个伪终端或终端。
  * -d: 后台启动
  * -P: 将容器内部使用的网络端口映射到我们使用的主机上。
  * -p: docker内部端口:外部端口
  * 输入exit退出终端
> * image: 镜像名称（centos:centos7）
> * command: 启动命令(/bin/echo "Hello World!")

* 进入已存在容器：docker attach containerId command
> 输入exit则退出容器
* 进入已存在容器：docker exec containerId command
* 导出容器：docker export containerId >xxxx.tar
* 导入本地容器：cat docker/ubuntu.tar | docker import - test/ubuntu:v1
* 导入外网容器：docker import http://example.com/exampleimage.tgz example/imagerepo
* 删除容器：docker rm -f 1e560fca3906
* 清空所有已停止的容器：docker container prune
* 查看某容器端口情况：docker port containerId
* 查看docker当前进程：docker top containerId
# Docker

## Docker简介

Docker是一个开源的应用容器引擎，让开发者可以打包应用及依赖包到一个可移植的镜像中，然后发布到任何流行的Linux或Windows机器上。使用Docker可以更方便地打包、测试以及部署应用程序。

## Docker环境安装

- 安装`yum-utils`；

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 为yum源添加docker仓库位置；

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

- 安装docker服务；

```
yum install docker-ce
```

- 启动docker服务。

```
systemctl start docker
```

## Docker镜像常用命令

### 搜索镜像

```
docker search java
```

![img](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwl3E8cVETIzWWyuK3KGwGvNMbeXicmWDFccAibvNFPbDuX0ht6iaz9eM6mmia2lYCeowjfZAJzAbq0N7g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 下载镜像

```
docker pull java:8
```

### 查看镜像版本

> 由于`docker search`命令只能查找出是否有该镜像，不能找到该镜像支持的版本，所以我们需要通过`Docker Hub`来搜索支持的版本。

- 进入`Docker Hub`的官网，地址：https://hub.docker.com
- 然后搜索需要的镜像：

![img](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwl3E8cVETIzWWyuK3KGwGvN2Ga1MJBVQ6k2Zib8Jsv46hMOIYBkcGk9A4yTpDWibSVjDwmKOFF7vBgQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 查看镜像支持的版本：

![img](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwl3E8cVETIzWWyuK3KGwGvNDY6905oa1MUibEnZYAAy7IeLlFj12vP0ibJibibic2aMg5A9vxibJJtaJsuA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 进行镜像的下载操作：

```
docker pull nginx:1.17.0
```

### 列出镜像

```
docker images
```

![img](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwl3E8cVETIzWWyuK3KGwGvN6Z7dPGhTY3WY87PwXTaXtoI1vzYoFKdszWv4d8siawR7zQSFAt9cKwg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 删除镜像

- 指定名称删除镜像：

```
docker rmi java:8
```

- 指定名称删除镜像（强制）：

```
docker rmi -f java:8
```

- 删除所有没有引用的镜像：

```
docker rmi `docker images | grep none | awk '{print $3}'`
```

- 强制删除所有镜像：

```
docker rmi -f $(docker images)
```

### 打包镜像

```
# -t 表示指定镜像仓库名称/镜像名称:镜像标签 .表示使用当前目录下的Dockerfile文件
docker build -t mall/mall-admin:1.0-SNAPSHOT .
```

## Docker容器常用命令

### 新建并启动容器

```
docker run -p 80:80 --name nginx \
-e TZ="Asia/Shanghai" \
-v /mydata/nginx/html:/usr/share/nginx/html \
-d nginx:1.17.0
```

- -p：将宿主机和容器端口进行映射，格式为：宿主机端口:容器端口；
- --name：指定容器名称，之后可以通过容器名称来操作容器；
- -e：设置容器的环境变量，这里设置的是时区；
- -v：将宿主机上的文件挂载到宿主机上，格式为：宿主机文件目录:容器文件目录；
- -d：表示容器以后台方式运行。

### 列出容器

- 列出运行中的容器：

```
docker ps
```

![img](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwl3E8cVETIzWWyuK3KGwGvNib8ub8SpPvN0g1cKSpw60rOBvUphqK7BG2vVcFeY5385dNwpCDLE4BQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 列出所有容器：

```
docker ps -a
```

![img](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwl3E8cVETIzWWyuK3KGwGvNMRK1hA4u3EEHPlrMbiaAK1mHEeSrMB6eVC36jCjqnnreZ8DzbuAxY4Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 停止容器

注意：`$ContainerName`表示容器名称，`$ContainerId`表示容器ID，可以使用容器名称的命令，基本也支持使用容器ID，比如下面的停止容器命令。

```
docker stop $ContainerName(or $ContainerId)
```

例如：

```
docker stop nginx
#或者
docker stop c5f5d5125587
```

### 强制停止容器

```
docker kill $ContainerName
```

### 启动容器

```
docker start $ContainerName
```

### 进入容器

- 先查询出容器的`pid`：

```
docker inspect --format "{{.State.Pid}}" $ContainerName
```

- 根据容器的pid进入容器：

```
nsenter --target "$pid" --mount --uts --ipc --net --pid
```

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### 删除容器

- 删除指定容器：

```
docker rm $ContainerName
```

- 按名称通配符删除容器，比如删除以名称`mall-`开头的容器：

```
docker rm `docker ps -a | grep mall-* | awk '{print $1}'`
```

- 强制删除所有容器；

```
docker rm -f $(docker ps -a -q)
```

### 查看容器的日志

- 查看容器产生的全部日志：

```
docker logs $ContainerName
```

- 动态查看容器产生的日志：

```
docker logs -f $ContainerName
```

### 查看容器的IP地址

```
docker inspect --format '{{ .NetworkSettings.IPAddress }}' $ContainerName
```

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### 修改容器的启动方式

```
# 将容器启动方式改为always
docker container update --restart=always $ContainerName
```

### 同步宿主机时间到容器

```
docker cp /etc/localtime $ContainerName:/etc/
```

### 指定容器时区

```
docker run -p 80:80 --name nginx \
-e TZ="Asia/Shanghai" \
-d nginx:1.17.0
```

### 查看容器资源占用状况

- 查看指定容器资源占用状况，比如cpu、内存、网络、io状态：

```
docker stats $ContainerName
```

- 查看所有容器资源占用情况：

```
docker stats -a
```
### 查看容器磁盘使用情况

```
docker system df
```

### 执行容器内部命令

```
docker exec -it $ContainerName /bin/bash
```

### 指定账号进入容器内部

```
# 使用root账号进入容器内部
docker exec -it --user root $ContainerName /bin/bash
```

### 查看所有网络

```
docker network ls
[root@local-linux ~]# docker network ls
NETWORK ID          NAME                     DRIVER              SCOPE
59b309a5c12f        bridge                   bridge              local
ef34fe69992b        host                     host                local
a65be030c632        none     
```

### 创建外部网络

```
docker network create -d bridge my-bridge-network
```

### 指定容器网络

```
docker run -p 80:80 --name nginx \
--network my-bridge-network \
-d nginx:1.17.0
```

## 修改镜像的存放位置

- 查看Docker镜像的存放位置：

```
docker info | grep "Docker Root Dir"
```

- 关闭Docker服务：

```
systemctl stop docker
```

- 先将原镜像目录移动到目标目录：

```
mv /var/lib/docker /mydata/docker
```

- 建立软连接：

```
ln -s /mydata/docker /var/lib/docker
```

- 再次查看可以发现镜像存放位置已经更改。
