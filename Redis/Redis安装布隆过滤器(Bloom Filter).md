# Redis安装布隆过滤器(Bloom Filter)
## 下载
```shell
wget https://github.com/RedisLabsModules/rebloom/archive/v2.2.6.tar.gz
```

## 解压
```shell
# 创建安装目录
mkdir /usr/local/redisbloom

# 解压
tar -zxvf v2.2.6.tar.gz

# 移动到安装目录
mv RedisBloom-2.2.6/ /usr/local/redisbloom/
```

## 编译
```shell
cd /usr/local/redisbloom/RedisBloom-2.2.6/
make
```

编译成功后，在 `RedisBloom-2.2.6` 目录下多了一个 `redisbloom.so` 文件。

## 集成 Redis
修改 **redis.conf** 配置文件
```shell
loadmodule /usr/local/redisbloom/RedisBloom-2.2.6/redisbloom.so
```

**重启 Redis！！！**

## 测试
```shell
192.168.78.100:6379> bf.add k1 1
(integer) 1
192.168.78.100:6379> bf.exists k1 1
(integer) 1
192.168.78.100:6379> bf.exists k1 2
(integer) 0
192.168.78.100:6379> 
```

**布隆过滤器的主要指令如下：**
-   bf.add 添加一个元素
-   bf.exists 判断一个元素是否存在
-   bf.madd 添加多个元素
-   bf.mexists 判断多个元素是否存在