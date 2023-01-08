# centos7 安装redis-6.0.9（多个Redis实例）

## 安装gcc

redis 是用 C 语言开发，安装之前先确认是否安装 gcc 环境，如果没有安装，执行以下命令进行安装：

```shell
yum install -y gcc
```

安装 6 版本的 redis，gcc 版本一定要 5.3 以上，查看 gcc 版本，centos7 默认安装4.8.5

```shell
[root@centos7 ~]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
目标：x86_64-redhat-linux
配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
线程模型：posix
gcc 版本 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
```

**升级gcc**

```shell
[root@centos7 ~]# yum -y install centos-release-scl && yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils && scl enable devtoolset-9 bash
```

`scl enable devtoolset-9 bash` 命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本，长期使用执行以下命令：

```shell
[root@centos7 ~]# echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

## 下载Redis & 解压

```shell
[root@centos7 download]# wget http://download.redis.io/releases/redis-6.0.9.tar.gz
[root@centos7 download]# tar -zxvf redis-6.0.9.tar.gz
```

## 编译

进入 redis 解压目录，执行编译命令

```shell
[root@centos7 download]# cd redis-6.0.9
[root@centos7 redis-6.0.9]# make

...

Hint: It's a good idea to run 'make test' ;)
make[1]: 离开目录“/download/redis-6.0.9/src”
[root@centos7 redis-6.0.9]#
```

如果在编译过程中出现错误，解决错误后再次编译，需要执行以下命令：

```shell
make distclean
```

## 安装

```shell
[root@centos7 redis-6.0.9]# make PREFIX=/usr/local/redis6 install
# PREFIX=/usr/local/redis6 为安装目录
```

## 配置环境变量

```shell
 vim /etc/profile
```

在文件末尾添加

```shell
export REDIS_HOME=/usr/local/redis6 # redis 安装目录
export PATH=$PATH:$REDIS_HOME/bin
```

刷新环境变量

```shell
source /etc/profile 
```

## 安装为系统服务

### 安装第一个实例

进入utils目录下，执行`install_server.sh`

```shell
[root@centos7 redis-6.0.9]# cd utils/
[root@centos7 utils]# ./install_server.sh 
Welcome to the redis service installer
This script will help you easily set up a running redis server

This systems seems to use systemd.
Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!
[root@centos7 utils]# 
```

执行`install_server.sh`时发生错误，解决方案：[Linux安装Redis6.0.9执行.install_server.sh报错](../问题总结/Linux安装Redis6.0.9执行.install_server.sh报错.md)

```shell
[root@centos7 utils]# ./install_server.sh 
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf] 
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log] 
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379] 
Selected default - /var/lib/redis/6379
Please select the redis executable path [/usr/local/redis6/bin/redis-server] 
Selected config:
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /var/log/redis_6379.log
Data dir       : /var/lib/redis/6379
Executable     : /usr/local/redis6/bin/redis-server
Cli Executable : /usr/local/redis6/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!
```

以上执行过程中，第五行`Please select the redis port for this instance: [6379] `，如果不输端口，默认为：6379

**查看 Redis 状态：**

```shell
[root@centos7 utils]# service redis_6379 status
Redis is running (14625)

```

### 安装第二个实例

**再次执行**​`./install_server.sh `​**命令：**

```shell
[root@centos7 utils]# ./install_server.sh 
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 6380
Please select the redis config file name [/etc/redis/6380.conf] 
Selected default - /etc/redis/6380.conf
Please select the redis log file name [/var/log/redis_6380.log] 
Selected default - /var/log/redis_6380.log
Please select the data directory for this instance [/var/lib/redis/6380] 
Selected default - /var/lib/redis/6380
Please select the redis executable path [/usr/local/redis6/bin/redis-server] 
Selected config:
Port           : 6380
Config file    : /etc/redis/6380.conf
Log file       : /var/log/redis_6380.log
Data dir       : /var/lib/redis/6380
Executable     : /usr/local/redis6/bin/redis-server
Cli Executable : /usr/local/redis6/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6380.conf => /etc/init.d/redis_6380
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!

```

以上执行过程中，第五行`Please select the redis port for this instance: [6379] `，输入端口：6380

**查看 Redis 状态：**

```shell
[root@centos7 utils]# service redis_6380 status
Redis is running (14709)

```

以此类推，可以安装多个服务实例。

‍
