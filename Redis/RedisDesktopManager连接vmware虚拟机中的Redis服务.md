# Redis Desktop Manager 连接vmware虚拟机中的Redis服务

### 一、环境

~~~
win10 + vmware 16 pro + centos7( 桥接模式 )

确保宿主机能够 ping 通虚拟机
~~~

### 二、修改 redis.conf  文件

修改后：

~~~shell
bind 192.168.0.120 #绑定虚拟机IP访问
port 6379          #端口号为6379
protected-mode no  #关闭保护模式
daemonize yes      #设为后台运行
requirepass 123456 #设置密码
~~~

登录 Redis，确定 Redis bind 的 IP 为虚拟机的 IP：

~~~shell
[root@centos7 bin]# ./redis-server redis.conf 
[root@centos7 bin]# ./redis-cli -h 192.168.0.120 -p 6379 -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.0.120:6379> config get bind
1) "bind"
2) "192.168.0.120"
192.168.0.120:6379> 
~~~

### 三、开启 6379 端口

~~~shell
[root@centos7 local]# firewall-cmd --query-port=6379/tcp 
no
[root@centos7 local]# firewall-cmd --zone=public --add-port=6379/tcp --permanent
success
[root@centos7 local]# firewall-cmd --reload 
success
[root@centos7 local]# firewall-cmd --query-port=6379/tcp
yes
[root@centos7 local]# 

firewall-cmd --query-port=6379/tcp                         # 查看指定端口是否开放
firewall-cmd --zone=public --add-port=6379/tcp --permanent # 添加6379端口
firewall-cmd --reload                                      # 重启防火墙
firewall-cmd --list-port                                   # 查看所有开放端口号

~~~

连接 Redis 。