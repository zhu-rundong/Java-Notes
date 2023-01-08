# 基础命令

## 启动

通过源码安装的redis

```shell
redis-server 配置文件路径
例：
redis-server /etc/redis/redis.conf
```

通过apt-get或者yum install安装的redis

```shell
/etc/init.d/redis-server start
/etc/init.d/redis-server restart
```

## 停止

通过源码安装的redis

```shell
redis-cli -h 127.0.0.1 -p 6379 shutdown                   # 未设置redis密码
redis-cli -h 127.0.0.1 -p 6379 -a password shutdown       # 设置redis密码
redis-cli -h 192.168.78.100 -p 6379 -a 123456 shutdown  
```

通过apt-get或者yum install安装的redis

```shell
/etc/init.d/redis-server stop
```

## 查看Redis服务状态

```shell
ps aux | grep redis-server
或
service redis_6379 status
```

## 查看redis版本

```shell
redis-server -v
```

## 连接redis客户端

```shell
redis-cli --raw
redis-cli --raw -h 127.0.0.1 -p 6379 -a password
```

‍
