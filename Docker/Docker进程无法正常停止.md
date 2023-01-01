# Docker进程无法正常停止

### 问题

执行`systemctl stop docker`，结果如下：

```shell
[root@centos7 ~]# systemctl stop docker
Warning: Stopping docker.service, but it can still be activated by:
  docker.socket
```

### 原因

这是因为除了`docker.service unit file`外，还有一个`docker.socket unit file`。警告意味着如果你试图停止`docker`服务，但`docker.socket unit file`还处于激活状态。

### 解决方法

#### 停止 docker.socket 服务

```shell
# 停止 docker 服务
systemctl stop docker.socket
systemctl stop docker.service
```

#### 删除 docker socket 文件

```shell
rm -rf /lib/systemd/system/docker.socket
```

‍
