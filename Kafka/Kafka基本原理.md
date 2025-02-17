# Kafka 基本原理

## 概述

Kafka 最初由 Linkedin 公司开发，使用 scala 语言编写，是一个分布式、支持分区的（partition）、多副本的（replica），基于 zookeeper 协调的分布式消息系统。

kafka 的最大特性是可以实时的处理大量数据以满足各种需求场景：比如基于 hadoop 的批处理系统、低延迟的实时系统、Storm/Spark 流式处理引擎，web/nginx 日志、访问日志，消息服务等等。

## 基本概念

### Broker

消息中间件处理节点，一个 Kafka 节点就是一个 broker，一个或者多个 Broker 可以组成一个 Kafka 集群。

### Topic

Kafka 根据 topic 对消息进行归类，发布到 Kafka 集群的每条消息都需要指定一个 topic。

### Producer

消息生产者，向 Broker 发送消息的客户端。

### Consumer

消息消费者，从 Broker 读取消息的客户端。

### ConsumerGroup

每个 Consumer 属于一个特定的 Consumer Group，一条消息可以被多个不同的 Consumer Group 消费，但是一个 Consumer Group 中只能有一个 Consumer 能够消费该消息。

**从一个较高的层面上来看，producer 通过网络（TCP 协议）发送消息到 Kafka 集群，然后 consumer 来进行消费，如下图：**

![image-20250216204619498](assets/image-20250216204619498.png)

**`可以这么来理解Topic，Partition和Broker`**

一个 topic，代表逻辑上的一个业务数据集，比如按数据库里不同表的数据操作消息区分放入不同 topic，订单相关操作消息放入订单 topic，用户相关操作消息放入用户 topic，对于大型网站来说，后端数据都是海量的，订单消息很可能是非常巨量的，比如有几百个 G 甚至达到 TB 级别，如果把这么多数据都放在一台机器上肯定会有容量限制问题，那么就可以在 topic 内部划分多个 partition 来分片存储数据，不同的 partition 可以位于不同的机器上，每台机器上都运行一个 Kafka 的进程 Broker。

**`为什么要对Topic下数据进行分区存储？`**

1. commit log 文件会受到所在机器的文件系统大小的限制，分区之后可以将不同的分区放在不同的机器上，相当于对数据做了**分布式存储**，理论上一个 topic 可以处理任意数量的数据。
2. 为了 **提高并行度**。





