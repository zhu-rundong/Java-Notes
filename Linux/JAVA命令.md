# JAVA 命令

## 查看已安装的 JDK 路径

```shell
# 查找所有已安装的 Java 路径
sudo alternatives --list | grep java
```

##  管理默认 JDK

### 注册 JDK 到 alternatives 系统

```shell
# 注册 OpenJDK 11
sudo alternatives --install /usr/bin/java java /usr/lib/jvm/java-11-openjdk-11.0.22.0.7-1.el7_9.x86_64/bin/java 2

# 注册 OpenJDK 8
sudo alternatives --install /usr/bin/java java /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.382.b05-1.el7_9.x86_64/jre/bin/java 1
```

- 参数说明：
  - `/usr/bin/java`：符号链接的目标路径。
  - `2` 和 `1`：优先级（数值越大优先级越高）。

### 切换默认 JDK

```shell
sudo alternatives --config java
```

按提示输入对应编号（例如选择 `2` 表示 OpenJDK 11）。