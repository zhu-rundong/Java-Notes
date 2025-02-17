# Kakfa命令

## 启动命令

```shell
# 启动kafka，运行日志在logs目录的server.log文件里
./bin/kafka-server-start.sh -daemon ./config/server.properties   #后台启动，不会打印日志到控制台
# 或者
./bin/kafka-server-start.sh config/server.properties &
```

## 停止kafka

```shell
./bin/kafka-server-stop.sh
```









