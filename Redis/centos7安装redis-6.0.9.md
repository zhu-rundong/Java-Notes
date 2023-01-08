# centos7 安装 redis-6.0.9

### 一、安装 gcc

 redis 是用 C 语言开发，安装之前先确认是否安装 gcc 环境，如果没有安装，执行以下命令进行安装：

~~~sh
[root@centos7 local]# yum install -y gcc
~~~

安装 6 版本的 redis，gcc 版本一定要 5.3 以上，查看 gcc 版本，centos7 默认安装 4.8.5

~~~shell
[root@centos7 local]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
目标：x86_64-redhat-linux
配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
线程模型：posix
gcc 版本 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
[root@centos7 local]# 
~~~

**升级 gcc：**

~~~shell
[root@centos7 local]# yum -y install centos-release-scl && yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils && scl enable devtoolset-9 bash

# scl enable devtoolset-9 bash 命令启用只是临时的，退出 shell 或重启就会恢复原系统 gcc 版本，长期使用：
[root@centos7 local]# echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
~~~

### 二、下载 Redis & 解压

~~~shell
[root@centos7 local]# wget http://download.redis.io/releases/redis-6.0.9.tar.gz
[root@centos7 local]# tar -zxvf redis-6.0.9.tar.gz
~~~

### 三、编译 & 安装

创建 redis 安装目录
~~~shell
[root@centos7 local]# mkdir /usr/local/redis-master
~~~

cd 到 redis 解压目录，编译构建 redis
~~~shell
[root@centos7 local]# cd redis-6.0.9/
[root@centos7 local]# make PREFIX=/usr/local/redis-master install
~~~

### 四、启动服务

将 redis.conf 文件从解压的文件夹中复制到 user/local/redis-master/bin 下

~~~shell
[root@centos7 local]# cp /usr/local/redis-6.0.9/redis.conf /usr/local/redis-master/bin
~~~

进入 bin 目录，编辑 redis.conf
~~~shell
daemonize yes        #设为后台运行
requirepass 123456   #设置密码
~~~

启动 & 登录
~~~shell
[root@centos7 bin]# ./redis-server redis.conf
[root@centos7 bin]# ./redis-cli -h 127.0.0.1 -p 6379 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379>
127.0.0.1:6379> keys *
(empty array)
127.0.0.1:6379>
~~~
