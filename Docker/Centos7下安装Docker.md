# centos7下安装Docker

## 环境检查

Docker 需要 CentOS 6.5 (64-bit) 或更高的版本，系统内核版本为 3.10 及以上。

**查看内核版本：**

```shell
[root@centos7 ~]# uname -srm
Linux 3.10.0-1160.45.1.el7.x86_64 x86_64

3 - 内核版本.
10 - 主修订版本.
0-1160 - 次要修订版本.
```

## yum源更新

```shell
sudo yum update
```

## 安装yum-utils&驱动依赖

yum-util 提供 yum-config-manager 功能，主要用来设置阿里云 yum 源，另外两个是devicemapper驱动依赖

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 设置yum源为阿里云

```shell
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 更新yum缓存

```shell
sudo yum makecache fast
```

## 安装docker

```shell
sudo yum install -y docker-ce
```

## 查看docker版本

```shell
sudo docker -v
sudo docker version
```

## 开机启动

```shell
sudo systemctl enable docker
```

## 卸载docker

```shell
#停止容器
sudo systemctl stop docker
#移除之前的文件
sudo yum remove docker \
        docker-client \
        docker-client-latest \
        docker-common \
        docker-latest \
        docker-latest-logrotate \
        docker-logrotate \
        docker-selinux \
        docker-engine-selinux \
        docker-engine \
        docer-io
#删除目录
sudo rm -rf /etc/systemd/system/docker.service.d
sudo rm -rf /var/lib/docker
sudo rm -rf /var/run/docker
```

‍
