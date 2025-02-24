# centos7 安装 Zookeeper

## 下载

从 [Zookeeper 官网](http://zookeeper.apache.org/releases.html) 下载，并上传到 Linux 服务器，或者使用 wget 在线下载

```shell
wget https://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.5.8/apache-zookeeper-3.5.8-bin.tar.gz
```

将下载的 `Zookeeper` 压缩包上传到 Linux 服务器

## 解压

```shell
tar -zxvf apache-zookeeper-3.5.8-bin.tar.gz   # 解压
mv apache-zookeeper-3.5.8-bin /usr/local/zookeeper  # 移动到安装目录
```

## 编辑配置文件

复制 zoo_sample.cfg 文件的并命名为为 zoo.cfg

```shell
cd apache-zookeeper-3.5.8-bin/
cp conf/zoo_sample.cfg conf/zoo.cfg
```

修改 `zoo.cfg` 文件，主要是修改 `dataDir` 和 `dataLogDir` 的路径

```shell
dataDir=/usr/local/zookeeper/apache-zookeeper-3.5.8-bin/data
dataLogDir=/usr/local/zookeeper/apache-zookeeper-3.5.8-bin/logs
```

## 配置环境变量

```shell
 vim /etc/profile
```

在文件末尾添加

```shell
export ZK_HOME=/usr/local/zookeeper/apache-zookeeper-3.5.8-bin
export PATH=$ZK_HOME/bin:$PATH
```

## 刷新环境变量

```shell
source /etc/profile 
```

## 启动 zookeeper 服务

进入 bin 目录

```shell
[root@centos7 bin]# ./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/apache-zookeeper-3.5.8-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

`zookeeper` 相关命令

```shell
./zkServer.sh start     # 启动
./zkServer.sh stop      # 停止
./zkServer.sh restart   # 重启
./zkServer.sh status    # 查看状态
./zkCli.sh              # 启动客户端
```
