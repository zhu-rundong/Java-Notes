# Maven基础配置

## 仓库配置

### 远程仓库

不在本机中的一切仓库，都是远程仓库：分为中央仓库和本地私服仓库。默认的远程仓库使用的是Apache提供的中央仓库：https://mvnrepository.com/

### 本地仓库

本地仓库是开发者本地电脑中的一个目录，用来缓存远程下载，包含你尚未发布的临时构件。默认的本地仓库是${user.home}/.m2/repository。用户可使用`settings.xml`文件修改本地仓库。具体内容如下：

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository 
  -->
  <localRepository>本机中具体仓库地址</localRepository>

```

### 镜像仓库

如果仓库A可以提供仓库B存储的所有内容，那么就可以认为A是B的一个镜像。例如：在国内直接连接中央仓库下载依赖，由于一些特殊原因下载速度非常慢。这时，我们可以使用阿里云提供的镜像http://maven.aliyun.com/nexus/content/groups/public/来替换中央仓库http://repol.maven.org/maven2/。修改maven的`setting.xml`文件，具体内容如下：

```xml
<mirror> 
        <!-- 指定镜像ID（可自定义） -->
        <id>nexus-aliyun</id> 
        <!-- 匹配中央仓库（阿里云的仓库名称，不可以自定义，必须这么写）-->
        <mirrorOf>central</mirrorOf>
        <!-- 指定镜像名称（可自定义）  -->   
        <name>Nexus aliyun</name> 
        <!-- 指定镜像路径（镜像地址） -->
        <url>http://maven.aliyun.com/nexus/content/groups/public</url> 
</mirror>
```

### 仓库优先级问题

```
                                 |----->没有配置镜像仓库---->默认中央仓库
本地仓库----配置文件中的指定的仓库----
                                 |----->配置了镜像仓库---->到镜像仓库找---->默认中央仓库
```

## JDK配置

当你的idea中有多个jdk的时候，就需要指定你编译和运行的jdk，在`settings.xml`中配置：

```xml
<profile>
    <!-- 告诉maven我们用jdk1.8 -->
    <id>jdk-1.8</id>
    <!-- 开启JDK的使用 -->
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <!-- 配置编译器信息 -->
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
    </profile>
```

## Maven工程类型

1. **POM工程**：POM工程是逻辑工程。用在父级工程或聚合工程中。用来做jar包的版本控制
2. **JAR工程**：将会打包成jar，用作jar包使用。即常见的本地工程 ----> Java Project
3. **WAR工程**：将会打包成war，发布在服务器上的工程

## Maven项目结构

```
--MavenDemo 项目名
  --.idea 项目的配置，自动生成的，无需关注。
  --src
    -- main 实际开发内容
         --java 写包和java代码，此文件默认只编译.java文件
         --resource 所有配置文件。最终编译把配置文件放入到classpath中。
    -- test  测试时使用，自己写测试类或junit工具等
        --java 储存测试用的类
  pom.xml Maven的基础配置文件，配置项目和项目之间关系，包括配置依赖关系等等
```
