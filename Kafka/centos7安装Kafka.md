## 环境准备

由于 Kafka 是用 Scala 语言开发的，运行在 JVM 上，因此在安装 Kafka 之前需要先安装 JDK。

在 CentOS 7 安装 JDK 可参考：[Linux 安装 JDK](JAVA/JAVA基础/Linux安装JDK.md)

另外，`Kafka` 依赖 `zookeeper`，所以还需要先安装 zookeeper。`ZooKeeper` 安装可参考：[centos7 安装 Zookeeper](../Zookeeper/centos7安装Zookeeper.md)

## 下载

从 [官网](https://kafka.apache.org/downloads.html) 下载 Kafka，这里下载的版本是：kafka_2.13-2.6.0.tgz（ 2.13 是 scala 的版本，2.6.0 是 kafka 的版本）。

将下载的 `Kafka` 压缩包上传到 Linux 服务器

## 解压

```shell
tar -zxvf kafka_2.13-2.6.0.tgz         # 解压
mkdir /usr/local/kafka                 # 创建安装目录
mv kafka_2.13-2.6.0 /usr/local/kafka/  # 移动到安装目录
```

## 编辑配置文件

修改配置文件 config/server.properties

```shell
# kafka 部署的机器 ip 和提供服务的端口号
listeners=PLAINTEXT://192.168.78.100:9092
# kafka 的消息存储文件
log.dirs=/usr/local/kafka/kafka_2.13-2.6.0/kafka-logs
# kafka 连接 zookeeper 的地址
zookeeper.connect=192.168.78.100:2181
```

## 配置环境变量

```shell
 vim /etc/profile
```

在文件末尾添加

```shell
export KAFKA_HOME=/usr/local/kafka/kafka_2.13-2.6.0
export PATH=$KAFKA_HOME/bin:$PATH
```

## 刷新环境变量

```shell
source /etc/profile 
```

## 启动

```shell
# -daemon 表示以后台进程运行，否则 ssh 客户端退出后，就会停止服务
bin/kafka-server-start.sh -daemon config/server.properties
# 或者
bin/kafka-server-start.sh config/server.properties &

# 停止 kafka
bin/kafka-server-stop.sh
```

## 查看是否启动成功

```shell
ps aux|grep Kafka
# 或者
jps
```

查看是否有 Kafka 进程

