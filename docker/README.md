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