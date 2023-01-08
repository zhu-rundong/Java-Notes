# Linux安装Redis6.0.9执行.install_server.sh报错 

Linux 安装Redis6.0.9时，执行`./install_server.sh`时报错：

```shell
[root@centos7 utils]# ./install_server.sh 
Welcome to the redis service installer
This script will help you easily set up a running redis server

This systems seems to use systemd.
Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!
```

解决方法：

```shell
[root@centos7 utils]# vi install_server.sh
```

注释以下代码：

```shell
#bail if this system is managed by systemd
#_pid_1_exe="$(readlink -f /proc/1/exe)"
#if [ "${_pid_1_exe##*/}" = systemd ]
#then
#       echo "This systems seems to use systemd."
#       echo "Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!"
#       exit 1
#fi

```

重新运行 `./install_server.sh`即可
