# Docker数据卷

### 数据卷概念

数据卷是存在于一个或多个容器中的特定文件或文件夹，这个文件或文件夹以独立于 docker 文件系统的形式存在于宿主机中。

### 为什么需要数据卷？

出于效率等一系列原因，docker 容器的文件系统在宿主机上存在的方式很复杂，这会带来下面几个问题：

* 不能在宿主机上很方便地访问容器中的文件
* 无法在多个容器之间共享数据
* 当容器删除时，容器中产生的数据将丢失

### 数据卷特点

其生存周期独立于容器的生存周期

### 使用场景

* 在多个容器之间共享数据，多个容器可以同时以只读或者读写的方式挂载同一个数据卷，从而共享数据卷中的数据
* 当宿主机不能保证一定存在某个目录或一些固定路径的文件时，使用数据卷可以规避这种限制带来的问题
* 当你想把容器中的数据存储在宿主机之外的地方时，比如远程主机上或云存储上
* 当你需要把容器数据在不同的宿主机之间备份、恢复或迁移时，数据卷是很好的选择

### 相关命令

```shell
docker volume 

ls                     列出所有的数据卷
prune                  删除所有未使用的volumes，并且有 -f 选项
create   volumeName    创建数据卷
inspect  volumeName    显示数据卷的详细信息
rm       volumeName    删除一个或多个未使用的volumes，并且有 -f 选项
```

**启动容器时创建并挂载数据卷**

```shell
docker run -d -p 80:80  
                        -v src（宿主机的目录）:dst（容器的目录）
                        -v 卷名:/data (第一次卷是空,会将容器的数据复制到卷中,如果卷里面有数据,把卷数据的挂载到容器中)
                        image_name:tag(版本)
```

**例如：挂载自定义的 ngnix 欢迎页**

```shell
# 在/download/index下有个自定义的index.html，启动容器时创建并挂载index文件夹

# 方式一
[root@centos7 volumes]# docker run -d -p 80:80 -v /download/index:/usr/share/nginx/html nginx

# 方式二

# 创建卷 mynginx
[root@centos7 volumes]# docker run -d -p 80:80 -v mynginx:/usr/share/nginx/html nginx

# 查看所有卷
[root@centos7 volumes]# docker volume ls
DRIVER    VOLUME NAME
local     mynginx

# 查看卷信息，第一次卷是空,会将容器的数据复制到卷中，nginx 显示默认欢迎页
[root@centos7 volumes]# docker volume inspect mynginx
[
    {
        "CreatedAt": "2021-12-06T21:17:10+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mynginx/_data",
        "Name": "mynginx",
        "Options": null,
        "Scope": "local"
    }
]
[root@centos7 volumes]# cd /var/lib/docker/volumes/mynginx/_data
[root@centos7 _data]# ls
50x.html  index.html

# 修改欢迎页
[root@centos7 _data]# echo 'my nginx index'>index.html

# 再起一个 nginx 容器，仍然使用 mynginx 卷，此时卷里面有数据，把卷数据的挂载到容器中。
# 访问欢迎页，显示修改后的“my nginx index”
[root@centos7 _data]# docker run -d -p 81:80 -v mynginx:/usr/share/nginx/html nginx
```
