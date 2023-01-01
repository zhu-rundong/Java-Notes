# Docker命令

### Docker 基础操作

**系统版本为：centos7.x**

#### 启动 Docker

```shell
systemctl start docker
```

#### 关闭 Docker

```shell
systemctl stop docker
```

#### 重启 Docker

```shell
systemctl restart docker  
```

#### 守护进程重启

```shell
systemctl daemon-reload
```

#### 查看是否启动成功

```shell
docker ps -a
```

#### 查看 Docker 运行状态

```shell
systemctl status docker
```

#### 查看 Docker 版本

```shell
docker version
```

#### 查看 Docker 系统信息

```shell
docker info
```

### 镜像操作

#### 查看镜像列表

```shell
docker images
docker image ls
```

#### 所有镜像

```shell
docker search image_name
```

#### 下载镜像

```shell
docker pull image_name:tag(版本)
tag：版本，不加版本默认下载最新版本
```

#### 上传镜像

```shell
docker push image_name:tag(版本)
tag：版本，不加版本默认下载最新版本
```

#### 导出镜像

```shell
docker image save -o path+filename.tar.gz image_name
-o：导出路径
```

#### 导入镜像

```shell
docker image load -i path+filename.tar.gz
```

#### 删除镜像

```shell
docker image rm image_name:tag
tag：版本，不加版本默认删除最新版本
```

#### 删除没有名称和版本的镜像

```shell
docker image prune
```

#### 构建镜像

```shell
根据 Dockerfile 自动构建镜像
docker image build -t imageName:tag imagePath(镜像所在目录)

例：
docker image build -t centos7_nginx:v2 /opt/dockerfile/nginx
```

### 容器操作

> **Docker** 容器内的第一个进程（初始命令）必须一直处于前台运行的状态，否则这个容易就会处于退出状态。

#### 创建&启动容器并启动 bash（交互方式）

```shell
ducker run -it [--name myContainerName] image_name:tag /bin/bash

-it 分配交互式终端
--name 指定启动后的容器名字
/bin/bash 覆盖容器的初始命令

例:
docker run -it nginx:latest /bin/bash 创建并进入容器
```

#### 创建&启动容器

```shell
docker run -d -p 主机(宿主)端口:容器端口 image_name:tag(版本)

-d: 后台运行容器
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
-P: 随机分配主机端口

例:
docker run -d -p 80:80 nginx:latest
```

#### 连接到正在运行的容器

```shell
docker exec -it container_name/container_id /bin/bash

exec会分配一个新的终端（常用）
```

```shell
docker attach container_name/container_id
attach上去的容器必须正在运行，可以同时连接上同一个容器来共享屏幕

crtl + d 退出
crtl + p & crtl + q 静默退出
```

#### 查看容器

```shell
docker container ls (-a -l -q ) 不加参数，表示查看正在运行的容器
或
docker ps (-a -l -q )

-a  所有容器
-l  最近运行的容器
-q  静默模式，只输出容器id=
--no-trunc  显示容器全部信息
```

#### 启动容器

```shell
docker start container_name/container_id
```

#### 停止容器

```shell
docker stop container_name/container_id
```

#### 重启容器

```shell
docker restart container_name/container_id
```

#### 杀掉一个运行中的容器

```shell
docker kill container_name/container_id
```

#### 删除容器

```shell
docker rm container_name/container_id ......

-f  强制删除

删除所有已经停止的容器
docker rm $(docker ps -a -q)
```

#### 查看日志

```shell
docker logs [OPTIONS] CONTAINER

-f       跟踪实时日志
--since  显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
--tail   从日志末尾显示多少行日志， 默认是all
-t       显示时间戳


# 查看最后100行日志
docker logs -f -t --tail 100 CONTAINER_ID

# 查看指定时间后的日志
docker logs -f -t --since="2021-11-08" CONTAINER_ID

# 查看最近10分钟的日志
docker logs --since 10m CONTAINER_ID

# 最后100行转存到/data/logs01.log：
docker logs --tail 100 CONTAINER_ID > /data/logs01.log
```

#### 将容器保存为镜像

```shell
docker commit 容器id或容器的名字 新的镜像名字[:版本号可选]
```

### 网络访问

> **容器默认172网段**

#### 查看网络映射

```shell
iptables -t nat -L -n
```

#### 创建&启动容器（分配不同 IP、端口）

```shell
docker run -d -p 主机(宿主)端口:容器端口 image_name:tag(版本)
              -p ip:主机(宿主)端口:容器端口   多个容器使用一个主机端口(每个容器的IP、端口不同，但共用一个宿主机端口)
              -p ip::容器端口  随机分配主机端口
              -p 主机(宿主)端口:容器端口 -p 主机(宿主)端口:容器端口  指定多个端口
-d: 后台运行容器
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
```

### rootfs 命令

#### 容器与主机之间数据拷贝

```shell
docker cp src_path containerId/containerName:target_path  将主机src_path目录拷贝到容器的target_path目录下

docker cp containerId/containerName:target_path src_path  将容器的target_path目录拷贝到主机src_path目录下
```

‍
